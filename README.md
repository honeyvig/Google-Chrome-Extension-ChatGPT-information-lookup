# Google-Chrome-Extension-ChatGPT-information-lookup
1. Use the ChatGPT chrome extension to look up the search term from the "keywords" list.  It is VERY important that you are using the ChatGPT chrome extension and NOT chat GPT itself.  Each search must be a new inquiry into the ChatGPT search bar.

2. Capture ChatGPT's sources on the "Chat GPT sites searched" tab

3. Capture the full text of the chat GPT response on the "Full text" tab

4. Capture the firms mentioned in the "firms mentioned" tab.
-------------------
Creating a Google Chrome extension that integrates with the ChatGPT Chrome extension is possible. To achieve this, the extension will need to automate searching within ChatGPT, capturing relevant data (such as the sources, full text, and firms mentioned), and displaying it in separate tabs.

Here's a high-level overview of how to create such a Chrome extension:
Structure of the Extension:

    Manifest File (manifest.json): Defines the basic settings of the extension.
    Background Script (background.js): Handles logic for searching and managing extension events.
    Content Script (content.js): Interacts with the web page, capturing data from ChatGPT.
    Popup (popup.html and popup.js): Displays the information in the extension's UI.
    CSS Files: Styles the popup for user interface.

Step-by-Step Breakdown:
1. Manifest File (manifest.json):

This file defines permissions, background scripts, and popup for the extension.

{
  "manifest_version": 3,
  "name": "ChatGPT Data Capture Extension",
  "description": "Capture data from ChatGPT responses and sources.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "tabs"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["https://chat.openai.com/*"],  // Match ChatGPT's URL
      "js": ["content.js"]
    }
  ],
  "host_permissions": [
    "https://chat.openai.com/*"
  ],
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

2. Background Script (background.js):

This script will handle sending messages to the content script and manage data collection.

chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.local.set({ chatData: [] });  // Initialize storage for chat data
});

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "captureData") {
    let chatData = message.data;
    chrome.storage.local.get(["chatData"], function(result) {
      let existingData = result.chatData || [];
      existingData.push(chatData);
      chrome.storage.local.set({ chatData: existingData });
    });
  }
});

3. Content Script (content.js):

This script will capture the data from ChatGPT after a search has been made. It should be triggered after a new inquiry in ChatGPT.

// Listen for changes or updates in the ChatGPT page (e.g., when a response is received)
const observeChatResponse = () => {
  const chatContainer = document.querySelector('.result-container');  // This is a placeholder, adapt it to actual container
  if (chatContainer) {
    const responseText = chatContainer.innerText;  // Capture full text
    const sources = document.querySelector('.sources');  // Placeholder for source links
    const firms = extractFirmsFromResponse(responseText);

    // Send captured data to background.js for storage
    chrome.runtime.sendMessage({
      action: "captureData",
      data: {
        fullText: responseText,
        sources: sources ? sources.innerText : "No sources",
        firmsMentioned: firms
      }
    });
  }
};

// Extract firms mentioned in the response (simple example)
const extractFirmsFromResponse = (responseText) => {
  const firmPattern = /\b[A-Z][a-z]+(?:\s[A-Z][a-z]+)*\b/g;  // Simple pattern to match capitalized words (firms)
  const firms = responseText.match(firmPattern) || [];
  return Array.from(new Set(firms));  // Remove duplicates
};

// Monitor for changes in the page (new response)
const observer = new MutationObserver(observeChatResponse);
observer.observe(document.body, { childList: true, subtree: true });

4. Popup UI (popup.html):

This file provides a user interface where you can see the collected data.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ChatGPT Data Capture</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 300px;
      padding: 10px;
    }
    h1 {
      font-size: 18px;
    }
    .tab {
      margin-bottom: 10px;
      padding: 5px;
      background-color: #f4f4f4;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <h1>Captured ChatGPT Data</h1>
  <div id="tabs">
    <div class="tab" id="fullTextTab">Full Text: </div>
    <div class="tab" id="sourcesTab">Sources: </div>
    <div class="tab" id="firmsTab">Firms Mentioned: </div>
  </div>

  <script src="popup.js"></script>
</body>
</html>

5. Popup Script (popup.js):

This script will display the captured data in the popup.

// Load chat data from local storage and populate the UI
chrome.storage.local.get(["chatData"], function(result) {
  const chatData = result.chatData || [];

  if (chatData.length > 0) {
    const latestChat = chatData[chatData.length - 1];

    // Display full text
    document.getElementById('fullTextTab').innerText += latestChat.fullText || 'No data';
    
    // Display sources
    document.getElementById('sourcesTab').innerText += latestChat.sources || 'No sources';
    
    // Display firms mentioned
    document.getElementById('firmsTab').innerText += latestChat.firmsMentioned.join(", ") || 'No firms';
  } else {
    document.getElementById('fullTextTab').innerText = 'No data captured yet';
  }
});

Key Features:

    Data Capture: The extension captures data from the ChatGPT responses: full text, sources, and firms mentioned.
    Real-Time Updates: As the user interacts with ChatGPT, the extension monitors and captures new data.
    Storage: The extension uses chrome.storage to save the captured data.
    Popup UI: The extension displays the latest captured data in a clean, user-friendly popup.

Conclusion:

This extension automates the process of searching with ChatGPT, capturing relevant data, and providing it to users in an organized manner. By automating the data collection, the extension ensures users can retrieve insights from ChatGPT in a structured way, enhancing productivity and efficiency.
