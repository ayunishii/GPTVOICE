let recognizing = false;
let recognition;
let synth = window.speechSynthesis;
let speakOutput = true;

const outputDiv = document.getElementById("output");

function initializeRecognition() {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  recognition = new SpeechRecognition();
  recognition.continuous = false;
  recognition.interimResults = false;
  recognition.lang = 'auto'; // Will set later dynamically

  recognition.onstart = () => {
    recognizing = true;
    outputDiv.innerText = "Listening...";
  };

  recognition.onresult = async (event) => {
    const spokenText = event.results[0][0].transcript;
    outputDiv.innerText = `You said: ${spokenText}`;
    const reply = await fetchGPTReply(spokenText);
    outputDiv.innerText += `\nGPT: ${reply}`;
    if (speakOutput) speak(reply);
  };

  recognition.onerror = (event) => {
    outputDiv.innerText = `Error: ${event.error}`;
  };

  recognition.onend = () => {
    recognizing = false;
  };
}

async function fetchGPTReply(text) {
  try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer sk-...9koA"
          
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: text }]
      })
    });

    const data = await response.json();
    const reply = data.choices?.[0]?.message?.content || "No response.";
    return reply;
  } catch (err) {
    return "Failed to fetch GPT reply.";
  }
}

function speak(text) {
  const utterance = new SpeechSynthesisUtterance(text);
  synth.speak(utterance);
}

function startListening() {
  if (!recognition) initializeRecognition();
  if (!recognizing) recognition.start();
}

function stopListening() {
  if (recognizing) recognition.stop();
}

function toggleSpeech() {
  speakOutput = !speakOutput;
  alert("Voice output is now " + (speakOutput ? "enabled" : "disabled"));
}
