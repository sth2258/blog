---
title: "GitHub discussions training"
layout: post
date: 2024-10-09
---

I was recently presented with an interesting challenge. Lots of organizations are attempting to fine tune their local LLMs to make them expert systems, and make responses relevant for their particular organization. 

What about discussions? Architectural decision records? Issue discussions? Comments? Wikis? Organizations using GitHub to track these are in luck! Enter, me :)

Using Amazon Bedrock however, there are no direct connectors for GitHub, so using the GitHub API (both traditional and GraphQL), I've written a quick `pwsh` script to scrape this data and dump it to a format that can directly be used for fine-tuning your model. 

At the end, you are presented with a jsonl file, as well as an entire directory of wiki content that you can use to store inside your S3 bucket to be used for a fine-tuning task. 

```
# Step 1: Set up GitHub API details
$githubToken = ""  # Replace with your GitHub API token


# Step 1: Set GitHub Organization and Output File
$orgName = "orgnamehere"  # GitHub Organization name
$outputFile = Join-Path (Get-Location) "conversations.jsonl"  # OS-agnostic output file path
$wikiDir = Join-Path (Get-Location) "wikis"  # OS-agnostic path for storing cloned wikis

# Function to make GitHub GraphQL API requests
function Invoke-GitHubGraphQL {
    param (
        [string]$query
    )
    
    $headers = @{
        Authorization = "Bearer $githubToken"
        Accept        = "application/vnd.github.v4+json"
    }

    $body = @{
        query = $query
    }

    $response = Invoke-RestMethod -Uri "https://api.github.com/graphql" -Headers $headers -Method Post -Body ($body | ConvertTo-Json)
    return $response
}

# Function to clone Wiki repositories (without checking if it exists beforehand)
function Clone-Wiki {
    param (
        [string]$repoName
    )
    
    $repoWikiDir = Join-Path $wikiDir $repoName

    if (-not (Test-Path -Path $repoWikiDir)) {
        # Clone the wiki repository
        git clone "https://github.com/$orgName/$repoName.wiki.git" $repoWikiDir
        Write-Host "Cloned Wiki for repository: $repoName"
    } else {
        Write-Host "Wiki for repository: $repoName already exists."
    }
}

# Function to move all .md files to the main wikis directory and handle duplicates
function Move-MarkdownFiles {
    param (
        [string]$sourceDir
    )

    $allMarkdownFiles = Get-ChildItem -Path $sourceDir -Recurse -Filter "*.md"
    
    foreach ($file in $allMarkdownFiles) {
        $destPath = Join-Path $wikiDir $file.Name

        # Check if the destination file exists
        if (Test-Path -Path $destPath) {
            # Add a random number prefix to avoid overwriting
            $randomNumber = Get-Random -Minimum 1000 -Maximum 9999
            $newFileName = "$randomNumber-$($file.Name)"
            $destPath = Join-Path $wikiDir $newFileName
        }

        # Move the file
        Move-Item -Path $file.FullName -Destination $destPath
    }
}

# Function to remove empty directories (ignoring .git directories)
function Remove-EmptyDirectories {
    param (
        [string]$rootDir
    )
    
    $allDirectories = Get-ChildItem -Path $rootDir -Directory -Recurse
    
    foreach ($dir in $allDirectories) {
        # Skip .git directories
        if ($dir.Name -eq ".git") {
            continue
        }

        # Check if the directory is empty, including hidden files
        $filesInDir = Get-ChildItem -Path $dir.FullName -Force
        if ($filesInDir.Count -eq 0) {
            # Remove the directory if empty
            Remove-Item -Path $dir.FullName -Recurse
            Write-Host "Removed empty directory: $dir"
        }
    }
}

# Function to extract issues and their comments
function Extract-Issues {
    param (
        [string]$repo
    )
    
    $query = @"
    {
      repository(owner: "$orgName", name: "$repo") {
        issues(first: 100) {
          edges {
            node {
              id
              number
              title
              body
              url
              comments(first: 100) {
                edges {
                  node {
                    id
                    body
                    author {
                      login
                    }
                    createdAt
                  }
                }
              }
            }
          }
        }
      }
    }
"@

    # Fetch issues using GraphQL
    $response = Invoke-GitHubGraphQL -query $query

    $issues = @()

    # Process issues
    if ($response.data.repository.issues.edges.Count -gt 0) {
        foreach ($issue in $response.data.repository.issues.edges) {
            $issueNode = $issue.node
            $issueData = @{
                repo_name = $repo
                issue_id = $issueNode.id
                issue_url = $issueNode.url
                issue_title = $issueNode.title
                issue_body = $issueNode.body
                comments = @()
            }

            # Process issue comments
            if ($issueNode.comments.edges.Count -gt 0) {
                foreach ($comment in $issueNode.comments.edges) {
                    $commentNode = $comment.node
                    $commentData = @{
                        comment_id = $commentNode.id
                        body = $commentNode.body
                        user = $commentNode.author.login
                        created_at = $commentNode.createdAt
                    }
                    $issueData.comments += $commentData
                }
            }

            $issues += $issueData
        }
        
        # Write issues to JSONL file for LLM training
        foreach ($issue in $issues) {
            $jsonLine = $issue | ConvertTo-Json
            Add-Content -Path $outputFile -Value $jsonLine
        }
    }
}

# Step 2: Get all repositories in the organization
$repos = gh repo list $orgName --limit 1000 --json name --jq '.[].name'

# Step 3: Loop through each repository to fetch discussions, issues, and clone wikis
$conversations = @()

foreach ($repo in $repos) {
    Write-Host "Processing repository: $repo"

    # GraphQL query to get discussions and their comments
    $query = @"
    {
      repository(owner: "$orgName", name: "$repo") {
        discussions(first: 100) {
          edges {
            node {
              id
              title
              body
              url
              comments(first: 100) {
                edges {
                  node {
                    id
                    body
                    author {
                      login
                    }
                    createdAt
                  }
                }
              }
            }
          }
        }
      }
    }
"@

    # Fetch discussions using GraphQL
    $response = Invoke-GitHubGraphQL -query $query

    # Initialize array for storing conversations
    $conversations = @()
    
    # Process discussions
    if ($response.data.repository.discussions.edges.Count -gt 0) {
        foreach ($discussion in $response.data.repository.discussions.edges) {
            $discussionNode = $discussion.node
            $conversation = @{
                repo_name      = $repo
                discussion_id  = $discussionNode.id
                discussion_url = $discussionNode.url
                title = $discussionNode.title
                body = $discussionNode.body
                comments = @()
            }

            # Process comments
            if ($discussionNode.comments.edges.Count -gt 0) {
                foreach ($comment in $discussionNode.comments.edges) {
                    $commentNode = $comment.node
                    $commentData = @{
                        comment_id = $commentNode.id
                        body = $commentNode.body
                        user = $commentNode.author.login
                        created_at = $commentNode.createdAt
                    }
                    $conversation.comments += $commentData
                }
            }

            $conversations += $conversation
        }

        # Write conversations to JSONL file for LLM training
        foreach ($conversation in $conversations) {
            $jsonLine = $conversation | ConvertTo-Json
            Add-Content -Path $outputFile -Value $jsonLine
        }
    }

    # Extract issues and their comments
    Extract-Issues -repo $repo

    # Clone Wiki repository
    Clone-Wiki -repoName $repo
}

# Step 4: Move all .md files to the wikis directory and handle duplicates
Move-MarkdownFiles -sourceDir $wikiDir

# Step 5: Remove empty directories (ignore .git)
Remove-EmptyDirectories -rootDir $wikiDir

Write-Host "Conversation data extracted, issues extracted, markdown files moved, and empty directories removed."
```