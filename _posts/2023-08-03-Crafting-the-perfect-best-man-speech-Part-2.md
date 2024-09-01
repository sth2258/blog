---
title: "Crafting the perfect best man Speech: Part 2 - Preprocessing"
layout: post
date: 2023-08-03
---

Building Robodave wasn’t just about using advanced AI tools; it was also about dealing with the mountains of data needed to make the AI work. My real challenge here was turning raw, unstructured data from various sources into something that could be used to train a model. This  post will dive into the preprocessing journey, explaining how I transformed chat logs and phone call recordings into a usable format for fine-tuning Robodave.
<!--more-->
## The Data Sources

To make Robodave sound and think like Dave, I needed data—lots of it. Here’s what I used:
- Chat Conversations: These were the primary data source. Over the years, Dave and I had countless chats through Microsoft Teams. These conversations would form the basis of the AI's communication style.
- Phone Call Recordings: Dave’s startup uses Fireflies AI to record and transcribe calls. These audio files were invaluable for training the voice model, ensuring that Robodave would not only speak like Dave but also sound like him.

## Retrieving and Preparing Chat Data

The first hurdle was getting my hands on all our chat logs. The data was captured in Microsoft Teams, so using the Office 365 Legal Hold feature could be used to extract the data. 

### Step 1: Downloading Chat Logs
- Place the user under Office 365's Legal Hold -- then using the Discovery features, download the chat conversations between Dave and myself. The extracted messages were stored as email messages in a PST archive. While PST files are commonly used for storing Outlook data, they're not exactly designed for easy data extraction.

### Step 2: Parsing PST Files
- To get the chat data out of the PST files, I turned to PowerShell. With some scripting, I was able to parse the PST files and extract the relevant messages. Here’s a snippet of what that looked like:

```
# Load the Outlook COM Object
$Outlook = New-Object -ComObject Outlook.Application

# Path to the PST file
$pstFilePath = "pstexport.pst"

# Name of the folder to parse (e.g., "Inbox")
$folderName = "FolderName"

# Output CSV file path
$outputCsvPath = "output.csv"

# Add PST file to Outlook's profile
$namespace = $Outlook.GetNamespace("MAPI")
$namespace.AddStore($pstFilePath)

# Get the folder object
$pstRootFolder = $namespace.Folders
$targetFolder = $pstRootFolder.Folders.Item($folderName)

# Array to store message data
$messageData = @()

# Function to parse messages in a folder
function Parse-Folder($folder) {
    foreach ($item in $folder.Items) {
        if ($item -is [Microsoft.Office.Interop.Outlook.MailItem]) {
            $messageDetails = [PSCustomObject]@{
                Subject      = $item.Subject
                Sender       = $item.SenderName
                ReceivedTime = $item.ReceivedTime
                Body         = $item.Body
            }
            $messageData += $messageDetails
        }
    }

    # Recursively parse subfolders
    foreach ($subFolder in $folder.Folders) {
        Parse-Folder -folder $subFolder
    }
}

# Start parsing the target folder
Parse-Folder -folder $targetFolder

# Export the data to a CSV file
$messageData | Export-Csv -Path $outputCsvPath -NoTypeInformation

# Remove the PST file from Outlook's profile
$namespace.RemoveStore($pstRootFolder)

# Notify the user
Write-Host "Messages exported to $outputCsvPath"
```

### Step 3: Formatting for OpenAI

OpenAI requires training data in a specific format: JSONL (JSON Lines) with "prompt" and "completion" pairs. In this case, the "prompt" would be a message initiated by either Dave or me, and the "completion" would be the corresponding response.

```
function cleanbody{
    return $args[0] -replace "`n","" -replace "`r",""
}

$content = Import-Csv .\ToOrFromSteveOnly.csv
$prompt = ""
$completion = ""
$objects = @()
$onSteve = $true; #force to start with a Steve start
for($i=0; $i -le $content.Count; $i++){
    $line = $content[$i];
    if($line.SenderName -eq "Steve Haber"){
        if($onSteve -eq $true){
            # I was already on steve - Just append my message to the prompt
            $prompt += (cleanbody $line.Body) + " "
            
        }else {
            # I was not on Steve - close the completion and end the document, clear it all
            #write-host "Prompt: $prompt"
            #write-host "Completion: $completion"
            $completion += "###" #stop sequence
            $prompt += "###" #stop sequence
            $entry = @{prompt = $prompt; completion = $completion}
            $objects +=$entry

            $prompt = ""; 
            $completion = "";
            $prompt += (cleanbody $line.Body) + " "

        }
        $onSteve = $true;
    }
    else {
        # It's a Dave line
        if($onSteve -eq $true){
            # means I was previously on a Steve message, means this is purely set completion
            $completion = (cleanbody $line.Body);
        }
        else {
            # I was not on a steve line, meaning it's a repeat another dave line, append 
            $completion += (cleanbody $line.Body) + " ";

        }
        $onSteve = $false;
    }
}

$str = ""
$objects |
%{
    $str += ($_ | ConvertTo-Json  -Compress) + [System.Environment]::NewLine
} 

$str | out-file "trainingdata.jsonl"
```

## Training the Voice Model

The next challenge was to create a voice model that could accurately mimic Dave’s speech patterns. This required high-quality audio data, and luckily, Dave’s startup provided just that through their recorded phone calls.

### Step 1: Collecting Audio Data

With Dave’s consent, I accessed the call recordings stored by Fireflies AI. These recordings captured Dave’s tone, inflection, and typical speech patterns—essential elements for training the voice model. The trick for this is that Fireflies has not only call recordings, but also the transcripts - This is absolutely key as without this, we'd need to have Dave [record specific exerpts](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/how-to-custom-voice-training-data), which would give away the whole surprise. 

### Step 2: Training with Azure AI Speech Services

Using Azure AI Speech Services, I fed the collected audio data into the system to create a voice clone. The process involved training the model on hours of Dave’s voice, fine-tuning it to ensure that it could replicate his speech naturally.

## Challenges in Preprocessing

This entire preprocessing effort wasn’t without its difficulties:
- Data Extraction: Extracting chat logs from PST files was more complex than I anticipated. PowerShell scripting became crucial in overcoming this challenge.
- Data Formatting: Ensuring the data was in the correct format for OpenAI’s fine-tuning was another significant task. Every conversation had to be carefully structured to avoid any misinterpretation by the model.
- Audio Quality: The quality of audio recordings varied, requiring additional steps to clean and preprocess the data before it could be used for training.

## Conclusion

Preprocessing was an essential part of bringing Robodave to life. By carefully extracting, cleaning, and formatting the data, I laid the groundwork for the AI models that would eventually power Robodave. In the next post, I’ll detail how all this work came together, resulting in a memorable AI-powered moment during my best man speech.