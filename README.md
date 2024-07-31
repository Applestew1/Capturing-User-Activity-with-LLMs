# Capturing-User-Activity-with-LLMs
Usually, when you use a help menu in an application, there are some common problems: 
- You can’t find the answer to your question.
- You find an answer, but it is not very specific
# Description
The program uses OpenAI to help answer questions posed by users and provides clear answers using context from screenshots and OpenAI’s GPT-4o-mini.
- The program takes screenshots of the user’s application at 2-second intervals.
- When the user poses a question, OpenAI scans the last five screenshots taken. 
- The program then gives a short and concise answer to the question.
# Getting Started
# Dependencies
- Windows 10 or 11
- Python Version 3.12 or later
- API Key
# Installing
- Head to python.org and follow the steps to download python 3.12 or later.
- You can download the program by copying the code.
# Executing program
- Step 1: Run the program and type in the name of the window you want monitored. (Example: File Explorer)
- Step 2: Press the "Start Monitoring" button to take screenshots. Press the "Stop Monitoring" button to stop taking screenshots. The program will keep only the last five 
  screenshots. You should see up to the last 5 screenshots in your files.
- Step 3 : Enter a question. ( Example: Can you summarize what I’m doing ? )
- Step 4: Press the "Ask OpenAI" button to get a response.
# Help
- You might need to have the window open and not minimized
- Make sure you have your API Key in the code
- Make sure you typed in the exact name of the window you need monitored.

# Authors
- Nathan Chiu
- nathanchiu123@gmail.com
# Version History
- 0.1
   Initial Release
# License
- This project is licensed under the Nathan Chiu License - see the LICENSE.md file for details
# Acknowledgments
- OpenAI
