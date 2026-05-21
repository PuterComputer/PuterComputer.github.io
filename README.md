import React, { useState, useRef } from "react";

export default function App() {
  const [windows, setWindows] = useState({
    about: { open: true, z: 1, title: "readme", top: 80, left: 160 },
    synth: { open: false, z: 2, title: "sounds", top: 140, left: 200 },
    contact: { open: false, z: 3, title: "contactme", top: 200, left: 240 },
    inbox: { open: false, z: 4, title: "inbox.txt", top: 260, left: 280 }
  });
  const [maxZ, setMaxZ] = useState(4);
  
  const audioCtxRef = useRef(null);

  const [contactName, setContactName] = useState("");
  const [contactMsg, setContactMsg] = useState("");
  const [statusMessage, setStatusMessage] = useState("");
  const [receivedMessages, setReceivedMessages] = useState([]);

  const playSynthNote = (freq, type = 'sawtooth') => {
    try {
      if (!audioCtxRef.current) {
        audioCtxRef.current = new (window.AudioContext || window.webkitAudioContext)();
      }
      const ctx = audioCtxRef.current;
      if (ctx.state === 'suspended') ctx.resume();

      const osc = ctx.createOscillator();
      const gainNode = ctx.createGain();

      osc.type = type;
      osc.frequency.setValueAtTime(freq, ctx.currentTime);
      
      if (type === 'sawtooth') {
        osc.frequency.exponentialRampToValueAtTime(freq * 0.5, ctx.currentTime + 0.4);
      }

      gainNode.gain.setValueAtTime(0.25, ctx.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.6);

      osc.connect(gainNode);
      gainNode.connect(ctx.destination);

      osc.start();
      osc.stop(ctx.currentTime + 0.6);
    } catch (e) {
      console.error("erro", e);
    }
  };

  const synthKeys = [
    { label: "Bass", freq: 130.81, type: "sawtooth" },
    { label: "Drone", freq: 164.81, type: "triangle" },
    { label: "Retro", freq: 196.00, type: "square" },
    { label: "Buzzy thing", freq: 220.00, type: "sawtooth" },
    { label: "Quiet bell", freq: 261.63, type: "sine" },
    { label: "Chiptune", freq: 293.66, type: "square" },
    { label: "Soft", freq: 329.63, type: "triangle" },
    { label: "Xylophone ahh", freq: 392.00, type: "sine" },
  ];

  const bringToFront = (id) => {
    setMaxZ(prev => prev + 1);
    setWindows(prev => ({
      ...prev,
      [id]: { ...prev[id], z: maxZ + 1 }
    }));
  };

  const toggleWindow = (id) => {
    setWindows(prev => {
      const isOpen = prev[id].open;
      setMaxZ(maxZ + 1);
      return {
        ...prev,
        [id]: { ...prev[id], open: !isOpen, z: isOpen ? prev[id].z : maxZ + 1 }
      };
    });
  };

  const handleSendContact = (e) => {
    e.preventDefault();
    if (!contactName.trim() || !contactMsg.trim()) {
      setStatusMessage("Please write your name and message first!");
      return;
    }

    const newMsg = {
      name: contactName,
      text: contactMsg,
      time: new Date().toLocaleTimeString()
    };

    setReceivedMessages(prev => [newMsg, ...prev]);
    setStatusMessage("Message saved! Open inbox.txt to see it.");
    setContactName("");
    setContactMsg("");
  };

  const renderWindow = (id, content) => {
    if (!windows[id].open) return null;
    return (
      <div 
        onMouseDown={() => bringToFront(id)}
        style={{ 
          zIndex: windows[id].z,
          position: "absolute",
          top: `${windows[id].top}px`,
          left: `${windows[id].left}px`,
          width: "300px",
          backgroundColor: "#c0c0c0",
          border: "2px solid black",
          borderTopColor: "white",
          borderLeftColor: "white"
        }}
      >
        <div style={{ backgroundColor: "#000080", color: "white", padding: "2px 4px", display: "flex", justifyContent: "space-between", fontWeight: "bold" }}>
          <span>{windows[id].title}</span>
          <button 
            onClick={(e) => { e.stopPropagation(); toggleWindow(id); }}
            style={{ backgroundColor: "#c0c0c0", border: "1px solid black", color: "black", fontWeight: "bold", padding: "0 4px", cursor: "pointer" }}
          >
            X
          </button>
        </div>
        <div style={{ padding: "8px", fontSize: "14px", color: "black", backgroundColor: "#c0c0c0" }}>
          {content}
        </div>
      </div>
    );
  };

  return (
    <div style={{ minHeight: "100vh", backgroundColor: "#008080", fontFamily: "serif", position: "relative" }}>
      <h1 style={{ color: "white", margin: 0, padding: "20px" }}>PUTER COMPUTER</h1>
      <p style={{ color: "white", paddingLeft: "20px" }}>Click the buttons on the side to learn stuff about me.</p>

      <div style={{ padding: "20px", display: "flex", flexDirection: "column", gap: "10px", width: "120px" }}>
         <button onClick={() => toggleWindow('about')} style={{ padding: "5px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}>About Me</button>
         <button onClick={() => toggleWindow('synth')} style={{ padding: "5px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}>Synth</button>
         <button onClick={() => toggleWindow('contact')} style={{ padding: "5px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}>Contact Me</button>
         <button onClick={() => toggleWindow('inbox')} style={{ padding: "5px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}>Inbox</button>
      </div>

      {renderWindow("about", (
        <div>
          <p><b>Hello! I'm Puter Computer.</b></p>
          <p>I'm an upcoming coder and music artist (if I can be bothered?? lol). I previously worked in the Roblox game development industry but I have decided to move on to something else, due to the new updates on the platform. Thanks for reading</p>
          <p>Try out the buttons on the side :)</p>
        </div>
      ))}

      {renderWindow("synth", (
        <div>
          <p>Click buttons to make noise (loud)</p>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: "5px" }}>
            {synthKeys.map((key, i) => (
              <button 
                key={i} 
                onClick={() => playSynthNote(key.freq, key.type)}
                style={{ padding: "10px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}
              >
                {key.label}
              </button>
            ))}
          </div>
        </div>
      ))}

      {renderWindow("contact", (
        <div>
          <p><b>Send a Message</b></p>
          <form onSubmit={handleSendContact}>
            <div style={{ marginBottom: "8px" }}>
              <label style={{ display: "block", marginBottom: "3px" }}>Your Name:</label>
              <input 
                type="text" 
                value={contactName}
                onChange={(e) => setContactName(e.target.value)}
                style={{ width: "100%", boxSizing: "border-box", border: "2px solid gray", borderBottomColor: "white", borderRightColor: "white", padding: "2px" }}
              />
            </div>
            <div style={{ marginBottom: "8px" }}>
              <label style={{ display: "block", marginBottom: "3px" }}>Message:</label>
              <textarea 
                value={contactMsg}
                onChange={(e) => setContactMsg(e.target.value)}
                rows={3}
                style={{ width: "100%", boxSizing: "border-box", border: "2px solid gray", borderBottomColor: "white", borderRightColor: "white", padding: "2px", resize: "none" }}
              />
            </div>
            <button 
              type="submit" 
              style={{ padding: "5px 10px", backgroundColor: "#c0c0c0", border: "2px solid black", borderTopColor: "white", borderLeftColor: "white", cursor: "pointer" }}
            >
              Send
            </button>
          </form>
          {statusMessage && (
            <p style={{ marginTop: "10px", color: "blue", fontWeight: "bold" }}>
              {statusMessage}
            </p>
          )}
        </div>
      ))}

      {renderWindow("inbox", (
        <div>
          <p><b>Messages Received</b></p>
          {receivedMessages.length === 0 ? (
            <p>No messages yet.</p>
          ) : (
            <div style={{ maxHeight: "200px", overflowY: "auto", border: "2px solid gray", backgroundColor: "white", padding: "4px" }}>
              {receivedMessages.map((m, idx) => (
                <div key={idx} style={{ borderBottom: "1px dashed black", paddingBottom: "5px", marginBottom: "5px" }}>
                  <span style={{ fontSize: "10px", color: "gray" }}>[{m.time}]</span> <b>{m.name}:</b>
                  <p style={{ margin: "2px 0 0 0" }}>{m.text}</p>
                </div>
              ))}
            </div>
          )}
        </div>
      ))}

    </div>
  );
}
