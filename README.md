# Complete Windows Guide: Extract All Conversations with "custom_gpt_name123" GPT

## What This Guide Does
This guide helps you extract ALL your conversations with a specific GPT (in your case "custom_gpt_name123") from your ChatGPT export and save them as a clean text file sorted by date.

## What You'll Need
- Your ChatGPT export ZIP file (from the email)
- Windows computer
- About 10-15 minutes

## Final Result
You'll get:
- A single text file with ALL conversations with your GPT
- Everything sorted by date
- Clean, readable format

---

# Step-by-Step Instructions

## Step 1: Get Your Export
1. Check your email for the ChatGPT export (usually arrives within 30 minutes)
2. Download the ZIP file to your Downloads folder
3. Note where you saved it (usually `C:\Users\YourName\Downloads`)

## Step 2: Extract the ZIP File
1. Go to your Downloads folder
2. Find the ChatGPT export ZIP file
3. Right-click on it → Select **"Extract All..."**
4. Click **"Extract"**
5. A new folder will appear with a name like `2024-01-15-...` 
6. Open this folder - you should see a file called `conversations.json`

## Step 3: Note Your Folder Path
1. Click in the address bar at the top of the File Explorer window
2. Select all the text (it will look like `C:\Users\YourName\Downloads\2024-01-15-...`)
3. Press **Ctrl+C** to copy this path
4. Open Notepad and paste it there temporarily (you'll need it soon)

## Step 4: Install Python
### Check if Python is already installed:
1. Press **Windows key**
2. Type `cmd` and press Enter
3. In the black window that appears, type exactly:
```
python --version
```
4. Press Enter

### If you see "Python 3.x.x":
- Great! Skip to Step 5

### If you see an error or "not recognized":
1. Open your web browser
2. Go to: https://www.python.org/downloads/
3. Click the big yellow button "Download Python 3.x.x"
4. Run the downloaded file
5. **IMPORTANT**: Check the box that says **"Add python.exe to PATH"** at the bottom
6. Click "Install Now"
7. When done, close and reopen the Command Prompt (cmd)
8. Type `python --version` again to verify it works

## Step 5: Create Your Work Folder
1. Press **Windows key**
2. Type `cmd` and press Enter
3. Copy and paste these commands one at a time (press Enter after each):

```
cd %USERPROFILE%\Desktop
```

```
mkdir GPT_Export
```

```
cd GPT_Export
```

## Step 6: Create the Python Script
1. Still in the Command Prompt, type:
```
notepad extract_gpt.py
```
2. Click "Yes" when it asks to create a new file
3. Copy ALL the text below and paste it into Notepad:

```python
import json
import os
from datetime import datetime
from pathlib import Path

# ===== SETTINGS - YOU MUST CHANGE THESE =====
# Replace this with YOUR actual folder path (the one you copied in Step 3)
EXPORT_FOLDER = r"C:\Users\YourName\Downloads\YOUR-EXPORT-FOLDER"

# The exact name of your GPT (copy it exactly from ChatGPT)
GPT_NAME = "custom_gpt_name123"
# ============================================

def extract_conversations():
    # Check if folder exists
    export_path = Path(EXPORT_FOLDER)
    json_file = export_path / "conversations.json"
    
    if not json_file.exists():
        print(f"ERROR: Cannot find conversations.json in {EXPORT_FOLDER}")
        print("Please check your EXPORT_FOLDER path in the script")
        return
    
    # Load the conversations
    print("Loading conversations...")
    with open(json_file, 'r', encoding='utf-8') as f:
        data = json.load(f)
    
    # Handle different export formats
    if isinstance(data, list):
        all_conversations = data
    elif isinstance(data, dict) and 'conversations' in data:
        all_conversations = data['conversations']
    else:
        print("ERROR: Unexpected file format")
        return
    
    print(f"Found {len(all_conversations)} total conversations")
    
    # Find matching conversations
    matching_convos = []
    
    for conv in all_conversations:
        # Check if this conversation mentions our GPT
        conv_str = json.dumps(conv).lower()
        if GPT_NAME.lower() in conv_str:
            # Get conversation info
            title = conv.get('title', 'Untitled')
            timestamp = conv.get('create_time', 0)
            
            # Extract messages
            messages = []
            mapping = conv.get('mapping', {})
            
            for node_id, node in mapping.items():
                if 'message' in node and node['message']:
                    msg = node['message']
                    if 'author' in msg and 'content' in msg:
                        role = msg['author'].get('role', '')
                        content = msg.get('content', {})
                        
                        # Extract text from parts
                        text_parts = []
                        if 'parts' in content:
                            for part in content['parts']:
                                if isinstance(part, str):
                                    text_parts.append(part)
                        
                        if text_parts and role in ['user', 'assistant']:
                            text = '\n'.join(text_parts)
                            msg_time = msg.get('create_time', timestamp)
                            messages.append({
                                'role': role,
                                'text': text,
                                'time': msg_time
                            })
            
            if messages:
                # Sort messages by time
                messages.sort(key=lambda x: x['time'])
                matching_convos.append({
                    'title': title,
                    'timestamp': timestamp,
                    'messages': messages
                })
    
    print(f"Found {len(matching_convos)} conversations with '{GPT_NAME}'")
    
    if not matching_convos:
        print("No matching conversations found. Check the GPT_NAME in the script.")
        return
    
    # Sort conversations by date
    matching_convos.sort(key=lambda x: x['timestamp'])
    
    # Create output file
    output_file = f"{GPT_NAME.replace(' ', '_')}_conversations.txt"
    
    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(f"All Conversations with {GPT_NAME}\n")
        f.write(f"Exported on: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        f.write(f"Total Conversations: {len(matching_convos)}\n")
        f.write("=" * 80 + "\n\n")
        
        for i, conv in enumerate(matching_convos, 1):
            # Write conversation header
            date_str = datetime.fromtimestamp(conv['timestamp']).strftime('%Y-%m-%d %H:%M:%S') if conv['timestamp'] else 'Unknown date'
            f.write(f"\n{'=' * 80}\n")
            f.write(f"CONVERSATION #{i}: {conv['title']}\n")
            f.write(f"Date: {date_str}\n")
            f.write(f"{'=' * 80}\n\n")
            
            # Write messages
            for msg in conv['messages']:
                role_label = "YOU" if msg['role'] == 'user' else "GPT"
                f.write(f"[{role_label}]:\n{msg['text']}\n\n")
                f.write("-" * 40 + "\n\n")
    
    print(f"\nSUCCESS! Created file: {output_file}")
    print(f"Location: {Path(output_file).absolute()}")
    print("\nYou can find this file on your Desktop in the GPT_Export folder")

if __name__ == "__main__":
    try:
        extract_conversations()
    except Exception as e:
        print(f"An error occurred: {e}")
        print("\nPlease check that:")
        print("1. Your EXPORT_FOLDER path is correct")
        print("2. The GPT_NAME matches exactly")
    
    input("\nPress Enter to close...")
```

## Step 7: Update Your Settings in the Script
**IMPORTANT - You must do this or it won't work:**

1. In the Notepad window, find this line near the top:
```python
EXPORT_FOLDER = r"C:\Users\YourName\Downloads\YOUR-EXPORT-FOLDER"
```

2. Replace it with YOUR actual folder path (the one you copied in Step 3)
   - Keep the `r"` at the beginning and the `"` at the end
   - Example: `EXPORT_FOLDER = r"C:\Users\John\Downloads\2024-01-15-abc123"`

3. Find this line:
```python
GPT_NAME = "custom_gpt_name123"
```

4. Make sure it matches EXACTLY how your GPT appears in ChatGPT
   - If unsure, leave it as "custom_gpt_name123" for now

5. Save the file: Press **Ctrl+S**
6. Close Notepad

## Step 8: Run the Script
1. Back in the Command Prompt window, type:
```
python extract_gpt.py
```
2. Press Enter
3. You should see messages like:
   - "Loading conversations..."
   - "Found X total conversations"
   - "Found X conversations with 'custom_gpt_name123'"
   - "SUCCESS! Created file: Metaphysical_Guide_conversations.txt"

4. Press Enter when it says "Press Enter to close..."

## Step 9: Find Your Output File
1. Go to your Desktop
2. Open the **GPT_Export** folder
3. You'll see a file named **Metaphysical_Guide_conversations.txt**
4. Double-click to open it in Notepad

---

# Troubleshooting

## "Cannot find conversations.json"
- Double-check your EXPORT_FOLDER path in the script
- Make sure you extracted the ZIP file first
- The path should point to the folder containing conversations.json, not the ZIP file

## "Found 0 conversations with 'custom_gpt_name123'"
Try these in order:
1. Try a partial name: Change to `GPT_NAME = "Metaphysical"`
2. Try lowercase: Change to `GPT_NAME = "custom_gpt_name123"`
3. Try just one word: Change to `GPT_NAME = "Guide"`

## "python is not recognized"
- You need to install Python (see Step 4)
- Make sure you checked "Add python.exe to PATH" during installation
- After installing, close and reopen Command Prompt

## The text file is too large or hard to read
- Open it in WordPad instead of Notepad (better for large files)
- Or open it in Microsoft Word

---

# Tips for Success

1. **Copy paths exactly** - One wrong character breaks everything
2. **Match the GPT name** - It needs to find the exact text
3. **Extract the ZIP first** - Don't try to read the ZIP directly
4. **Use the Desktop/GPT_Export folder** - Keeps everything organized

---

# What the Output Looks Like

Your text file will be organized like this:

```
All Conversations with custom_gpt_name123
Exported on: 2024-10-27 14:30:00
Total Conversations: 47
================================================================================

================================================================================
CONVERSATION #1: First Discussion about Consciousness
Date: 2024-01-15 09:23:45
================================================================================

[YOU]:
What is the nature of consciousness?

----------------------------------------

[GPT]:
Consciousness is one of the most profound mysteries...

----------------------------------------

[YOU]:
How does this relate to quantum mechanics?

----------------------------------------

[GPT]:
The relationship between consciousness and quantum mechanics...

----------------------------------------

================================================================================
CONVERSATION #2: Exploring Free Will
Date: 2024-01-16 14:15:22
================================================================================

[continuing for all conversations...]
```

---

# Next Steps

Once you have your text file, you can:
- Open it in Word for better formatting
- Search for specific topics (Ctrl+F)
- Copy sections to other documents
- Print it if needed
- Import into other analysis tools

---

# Need Help?

If something isn't working:
1. Double-check you followed each step exactly
2. Make sure your export folder path is correct
3. Try the troubleshooting section
4. The most common issue is the folder path - make sure it points to the extracted folder, not the ZIP

Good luck! This should give you a complete record of all your custom_gpt_name123 conversations in one clean file.
