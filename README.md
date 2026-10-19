
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>JARVIS AI</title>
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0; padding: 24px 16px; min-height: 100vh;
    background: #07111f; color: #e8f7ff;
    font-family: Arial, sans-serif; text-align: center;
  }
  h1 { color: #00e5ff; letter-spacing: 4px; }
  .orb {
    width: 145px; height: 145px; margin: 28px auto;
    border: 3px solid #00e5ff; border-radius: 50%;
    display: grid; place-items: center; font-size: 54px;
    box-shadow: 0 0 25px #00e5ff, inset 0 0 25px #007caa;
    animation: glow 2s infinite alternate;
  }
  @keyframes glow {
    from { box-shadow: 0 0 12px #00e5ff; }
    to { box-shadow: 0 0 35px #00e5ff, 0 0 55px #007caa; }
  }
  button, input {
    width: 100%; max-width: 360px; padding: 14px;
    margin: 7px auto; border-radius: 12px; font-size: 16px;
  }
  button {
    display: block; background: #071d30; color: #00e5ff;
    border: 1px solid #00e5ff; cursor: pointer;
  }
  input { background: #102033; color: white; border: 1px solid #00e5ff; }
  #status, #reply { margin: 18px auto; max-width: 400px; overflow-wrap: anywhere; }
  .hint { color: #a6c4d5; font-size: 14px; line-height: 1.6; }
</style>
</head>
<body>
<h1>JARVIS AI</h1>
<p>Your Personal Assistant</p>
<div class="orb">🤖</div>

<button id="listen">🎙️ Talk to Jarvis</button>
<button onclick="runCommand('open youtube')">▶ Open YouTube</button>
<button onclick="runCommand('open instagram')">📷 Open Instagram</button>
<button onclick="runCommand('what time is it')">🕒 Tell me the time</button>

<input id="command" placeholder="Type a command here">
<button onclick="runCommand(document.getElementById('command').value)">Run Command</button>

<div id="status">Status: Ready</div>
<div id="reply">Hello buddy! I am Jarvis.</div>

<p class="hint">
Try: “Open YouTube”, “Open Instagram”, “What time is it?”,
“Search YouTube for gaming”, “Call Mom”, or “Stop listening”.
</p>

<script>
const statusBox = document.getElementById("status");
const replyBox = document.getElementById("reply");

function speak(text) {
  replyBox.textContent = text;
  if ("speechSynthesis" in window) {
    speechSynthesis.cancel();
    const message = new SpeechSynthesisUtterance(text);
    message.lang = "en-IN";
    speechSynthesis.speak(message);
  }
}

function openLink(url) {
  window.location.href = url;
}

function runCommand(raw) {
  const text = String(raw || "").trim().toLowerCase();
  if (!text) {
    speak("Please say or type a command.");
    return;
  }

  statusBox.textContent = "Command: " + text;

  if (text.includes("stop listening")) {
    if (recognition) recognition.stop();
    speak("Okay buddy. I have stopped listening.");
    return;
  }

  if (text.includes("what time") || text === "time" ||
      text.includes("tell me the time")) {
    speak("The time is " + new Date().toLocaleTimeString("en-IN"));
    return;
  }

  if (text.includes("today's date") || text.includes("what is the date")) {
    speak("Today is " + new Date().toLocaleDateString("en-IN"));
    return;
  }

  if (text.includes("open youtube")) {
    speak("Opening YouTube.");
    openLink("https://www.youtube.com/");
    return;
  }

  if (text.includes("open instagram")) {
    speak("Opening Instagram.");
    openLink("https://www.instagram.com/");
    return;
  }

  if (text.includes("open chrome")) {
    speak("I will try to open a webpage in your browser.");
    openLink("https://www.google.com/");
    return;
  }

  if (text.includes("search youtube for")) {
    const query = text.split("search youtube for").slice(1).join(" ").trim();
    if (!query) {
      speak("Please say what you want to search for.");
      return;
    }
    speak("Searching YouTube for " + query);
    openLink("https://www.youtube.com/results?search_query=" + encodeURIComponent(query));
    return;
  }

  if (text.includes("call ") || text.startsWith("dial ")) {
    const numberMatch = text.match(/[+\d][\d\s-]{5,}/);
    let number = numberMatch ? numberMatch[0].replace(/[\s-]/g, "") : "";

    if (!number) {
      const answer = prompt(
        "Your browser cannot read your saved contacts. Enter the phone number to open in the dialer:"
      );
      if (!answer) {
        speak("Call cancelled.");
        return;
      }
      number = answer.replace(/[^\d+]/g, "");
    }

    if (!/^\+?\d{6,15}$/.test(number)) {
      speak("That number does not look valid. Please try again.");
      return;
    }

    if (confirm("Open your phone dialer with " + number + "? You can review the call before placing it.")) {
      speak("Opening the phone dialer. Please review and place the call yourself.");
      openLink("tel:" + number);
    } else {
      speak("Call cancelled.");
    }
    return;
  }

  if (text.includes("unlock") || text.includes("lock my phone") ||
      text.includes("turn off the screen")) {
    speak("Android does not allow this website to lock or unlock your phone. That needs supported Android system integration.");
    return;
  }

  if (text.includes("hello") || text.includes("hey jarvis")) {
    speak("Hello buddy! I am Jarvis, your personal assistant.");
    return;
  }

  speak("Sorry buddy, I do not know that command yet.");
}

let recognition = null;
const SpeechRecognition = window.SpeechRecognition ||
                          window.webkitSpeechRecognition;

if (SpeechRecognition) {
  recognition = new SpeechRecognition();
  recognition.lang = "en-IN";
  recognition.continuous = false;
  recognition.interimResults = false;

  recognition.onstart = () => statusBox.textContent = "Listening...";
  recognition.onresult = event => {
    const words = event.results[0][0].transcript;
    document.getElementById("command").value = words;
    runCommand(words);
  };
  recognition.onerror = event => {
    statusBox.textContent = "Voice error: " + event.error;
    speak("I could not hear that command. Please try again.");
  };
  recognition.onend = () => {
    if (statusBox.textContent === "Listening...") {
      statusBox.textContent = "Ready";
    }
  };

  document.getElementById("listen").onclick = () => {
    try { recognition.start(); }
    catch (error) { statusBox.textContent = "Please wait, then tap again."; }
  };
} else {
  document.getElementById("listen").onclick = () => {
    speak("Voice recognition is not supported in this browser. Please try a supported browser.");
  };
  statusBox.textContent = "Voice recognition may not be supported here.";
}
</script>
</body>
</html>
