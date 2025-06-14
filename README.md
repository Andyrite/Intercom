<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mobile Intercom System</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh; padding: 20px; color: white;
        }
        .container {
            max-width: 400px; margin: 0 auto;
            background: rgba(255, 255, 255, 0.1); backdrop-filter: blur(10px);
            border-radius: 20px; padding: 30px; box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }
        .header { text-align: center; margin-bottom: 30px; }
        .header h1 { font-size: 28px; margin-bottom: 10px; text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3); }
        .status {
            background: rgba(255, 255, 255, 0.2); padding: 15px; border-radius: 15px;
            margin-bottom: 20px; text-align: center; font-weight: 600;
        }
        .status.connected { background: rgba(34, 197, 94, 0.3); }
        .status.disconnected { background: rgba(239, 68, 68, 0.3); }
        .status.connecting { background: rgba(245, 158, 11, 0.3); }
        .setup-section { margin-bottom: 25px; }
        .input-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: 600; text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3); }
        input {
            width: 100%; padding: 15px; border: none; border-radius: 10px;
            background: rgba(255, 255, 255, 0.9); color: #333; font-size: 16px; font-weight: 500;
        }
        input::placeholder { color: #666; }
        .btn {
            width: 100%; padding: 15px; border: none; border-radius: 10px; font-size: 16px;
            font-weight: 600; cursor: pointer; transition: all 0.3s ease;
            text-transform: uppercase; letter-spacing: 1px;
        }
        .btn:disabled { opacity: 0.6; cursor: not-allowed; }
        .btn-primary {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white; box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
        }
        .btn-primary:hover:not(:disabled) {
            transform: translateY(-2px); box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
        }
        .btn-secondary { background: rgba(255, 255, 255, 0.2); color: white; margin-top: 10px; }
        .btn-secondary:hover { background: rgba(255, 255, 255, 0.3); }
        .audio-controls { display: flex; justify-content: center; align-items: center; margin: 20px 0; gap: 20px; }
        .mute-button {
            width: 80px; height: 80px; border-radius: 50%; border: none; cursor: pointer;
            font-size: 24px; transition: all 0.3s ease; display: flex; align-items: center;
            justify-content: center; box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
        .mute-button.unmuted { background: linear-gradient(135deg, #10b981 0%, #059669 100%); color: white; }
        .mute-button.muted {
            background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
            color: white; animation: pulse 2s infinite;
        }
        .mute-button:hover { transform: scale(1.05); }
        @keyframes pulse {
            0% { box-shadow: 0 4px 15px rgba(239, 68, 68, 0.4); }
            50% { box-shadow: 0 4px 25px rgba(239, 68, 68, 0.8); }
            100% { box-shadow: 0 4px 15px rgba(239, 68, 68, 0.4); }
        }
        .audio-status { text-align: center; font-weight: 600; margin-bottom: 10px; }
        .volume-indicator {
            width: 200px; height: 8px; background: rgba(255, 255, 255, 0.2);
            border-radius: 4px; overflow: hidden; margin: 0 auto;
        }
        .volume-bar {
            height: 100%; background: linear-gradient(90deg, #10b981, #f59e0b, #ef4444);
            width: 0%; transition: width 0.1s ease;
        }
        .participants {
            background: rgba(255, 255, 255, 0.1); border-radius: 15px;
            padding: 20px; margin-top: 20px;
        }
        .participants h3 { margin-bottom: 15px; text-align: center; }
        .participant {
            display: flex; align-items: center; justify-content: space-between;
            padding: 10px; background: rgba(255, 255, 255, 0.1);
            border-radius: 10px; margin-bottom: 10px;
        }
        .participant-info { display: flex; align-items: center; gap: 10px; }
        .speaking-indicator {
            width: 12px; height: 12px; border-radius: 50%; background: #10b981;
            opacity: 0; transition: opacity 0.3s ease;
        }
        .speaking-indicator.active { opacity: 1; animation: blink 1s infinite; }
        .muted-indicator {
            width: 12px; height: 12px; border-radius: 50%; background: #ef4444;
            opacity: 0; transition: opacity 0.3s ease;
        }
        .muted-indicator.active { opacity: 1; }
        @keyframes blink { 0%, 50% { opacity: 1; } 51%, 100% { opacity: 0.3; } }
        .controls { margin-top: 20px; text-align: center; }
        .hidden { display: none; }
        .instructions {
            background: rgba(255, 255, 255, 0.1); padding: 15px; border-radius: 10px;
            margin-bottom: 20px; font-size: 14px; line-height: 1.5;
        }
        .live-indicator {
            display: inline-block; width: 8px; height: 8px; background: #ef4444;
            border-radius: 50%; margin-right: 8px; animation: blink 1s infinite;
        }
        .firebase-warning {
            background: rgba(245, 158, 11, 0.3); border: 2px solid rgba(245, 158, 11, 0.5);
            padding: 15px; border-radius: 10px; margin-bottom: 20px; font-size: 14px; line-height: 1.5;
        }
        .connection-indicator {
            width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 8px;
        }
        .connection-indicator.good { background: #10b981; }
        .connection-indicator.poor { background: #f59e0b; }
        .connection-indicator.bad { background: #ef4444; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📞 Intercom</h1>
            <div class="status disconnected" id="status">Disconnected</div>
        </div>

        <div id="firebaseWarning" class="firebase-warning">
            <strong>⚠️ Firebase Configuration Required</strong><br>
            Please configure your Firebase project settings in the JavaScript code below.
        </div>

        <div id="setupSection" class="setup-section">
            <div class="instructions">
                <strong>Open Line Intercom:</strong><br>
                1. Create or join a room<br>
                2. Share the room code with others<br>
                3. Audio lines are always open<br>
                4. Click microphone to mute/unmute yourself
            </div>

            <div class="input-group">
                <label for="username">Your Name:</label>
                <input type="text" id="username" placeholder="Enter your name" maxlength="20">
            </div>

            <div class="input-group">
                <label for="roomCode">Room Code:</label>
                <input type="text" id="roomCode" placeholder="Enter room code or leave blank to create" maxlength="10">
            </div>

            <button class="btn btn-primary" id="joinButton" onclick="joinRoom()">Join Room</button>
        </div>

        <div id="intercomSection" class="hidden">
            <div class="audio-status" id="audioStatus">
                <span class="live-indicator"></span>Live Audio
            </div>
            
            <div class="audio-controls">
                <button class="mute-button unmuted" id="muteButton" onclick="toggleMute()">🎤</button>
            </div>
            
            <div class="volume-indicator">
                <div class="volume-bar" id="volumeBar"></div>
            </div>

            <div class="participants">
                <h3>Connected Users</h3>
                <div id="participantsList"></div>
            </div>

            <div class="controls">
                <button class="btn btn-secondary" onclick="leaveRoom()">Leave Room</button>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js';
        import { getDatabase, ref, push, onValue, off, set, onDisconnect, serverTimestamp } from 'https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js';
        // For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyCMBYGGQ3497RlqvpUMHfw1O58BakoDhY8",
  authDomain: "mobile-intercom-2025.firebaseapp.com",
  databaseURL: "https://mobile-intercom-2025-default-rtdb.firebaseio.com",
  projectId: "mobile-intercom-2025",
  storageBucket: "mobile-intercom-2025.firebasestorage.app",
  messagingSenderId: "1080571546583",
  appId: "1:1080571546583:web:b8968426c8fbe8f59f9992",
  measurementId: "G-N251G5Z157"
};

        // Check Firebase config
        const isConfigured = firebaseConfig.apiKey !== "your-api-key-here";
        if (!isConfigured) {
            console.warn('Firebase not configured. Running in demo mode.');
        } else {
            document.getElementById('firebaseWarning').style.display = 'none';
        }

        // Initialize Firebase
        let app, database;
        if (isConfigured) {
            try {
                app = initializeApp(firebaseConfig);
                database = getDatabase(app);
            } catch (error) {
                console.error('Firebase initialization failed:', error);
            }
        }

        class IntercomSystem {
            constructor() {
                this.localStream = null;
                this.peerConnections = new Map();
                this.participants = new Map();
                this.isConnected = false;
                this.isMuted = false;
                this.roomCode = null;
                this.username = null;
                this.userId = this.generateUserId();
                this.audioContext = null;
                this.analyser = null;
                this.volumeAnimation = null;
                this.roomRef = null;
                this.signalingListeners = new Map();

                this.rtcConfig = {
                    iceServers: [
                        { urls: 'stun:stun.l.google.com:19302' },
                        { urls: 'stun:stun1.l.google.com:19302' }
                    ]
                };

                this.isFirebaseConfigured = isConfigured && database;
            }

            generateUserId() {
                return Math.random().toString(36).substring(2, 15);
            }

            generateRoomCode() {
                return Math.random().toString(36).substring(2, 8).toUpperCase();
            }

            async startAudio() {
                if (this.localStream) return;
                
                try {
                    this.localStream = await navigator.mediaDevices.getUserMedia({
                        audio: {
                            echoCancellation: true,
                            noiseSuppression: true,
                            autoGainControl: true
                        }
                    });

                    this.setupVolumeMonitoring();
                } catch (error) {
                    console.error('Error starting audio:', error);
                    alert('Unable to access microphone. Please check your browser permissions.');
                    throw error;
                }
            }

            setupVolumeMonitoring() {
                if (!this.localStream) return;
                
                this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
                this.analyser = this.audioContext.createAnalyser();
                const source = this.audioContext.createMediaStreamSource(this.localStream);
                source.connect(this.analyser);
                
                this.analyser.fftSize = 256;
                const bufferLength = this.analyser.frequencyBinCount;
                const dataArray = new Uint8Array(bufferLength);
                
                const updateVolume = () => {
                    if (!this.localStream) return;
                    
                    this.analyser.getByteFrequencyData(dataArray);
                    const average = dataArray.reduce((a, b) => a + b) / bufferLength;
                    const volumePercent = (average / 255) * 100;
                    
                    if (!this.isMuted) {
                        document.getElementById('volumeBar').style.width = `${volumePercent}%`;
                    } else {
                        document.getElementById('volumeBar').style.width = '0%';
                    }
                    
                    this.volumeAnimation = requestAnimationFrame(updateVolume);
                };
                
                updateVolume();
            }

            async createPeerConnection(peerId) {
                const pc = new RTCPeerConnection(this.rtcConfig);
                
                if (this.localStream && !this.isMuted) {
                    this.localStream.getTracks().forEach(track => {
                        pc.addTrack(track, this.localStream);
                    });
                }

                pc.ontrack = (event) => {
                    const audioElement = document.createElement('audio');
                    audioElement.srcObject = event.streams[0];
                    audioElement.autoplay = true;
                    audioElement.id = `audio-${peerId}`;
                    document.body.appendChild(audioElement);
                };

                pc.onicecandidate = (event) => {
                    if (event.candidate && this.isFirebaseConfigured) {
                        this.sendSignal(peerId, {
                            type: 'ice-candidate',
                            candidate: event.candidate
                        });
                    }
                };

                pc.onconnectionstatechange = () => {
                    console.log(`Peer ${peerId} connection state:`, pc.connectionState);
                    if (pc.connectionState === 'failed') {
                        this.handlePeerDisconnection(peerId);
                    }
                };

                this.peerConnections.set(peerId, pc);
                return pc;
            }

            async handleNewPeer(peerId) {
                if (peerId === this.userId || this.peerConnections.has(peerId)) return;

                console.log('New peer joined:', peerId);
                const pc = await this.createPeerConnection(peerId);
                
                const offer = await pc.createOffer();
                await pc.setLocalDescription(offer);
                
                if (this.isFirebaseConfigured) {
                    this.sendSignal(peerId, {
                        type: 'offer',
                        offer: offer
                    });
                }
            }

            async handleSignal(fromPeerId, signal) {
                let pc = this.peerConnections.get(fromPeerId);
                
                if (!pc) {
                    pc = await this.createPeerConnection(fromPeerId);
                }

                try {
                    switch (signal.type) {
                        case 'offer':
                            await pc.setRemoteDescription(signal.offer);
                            const answer = await pc.createAnswer();
                            await pc.setLocalDescription(answer);
                            this.sendSignal(fromPeerId, {
                                type: 'answer',
                                answer: answer
                            });
                            break;

                        case 'answer':
                            await pc.setRemoteDescription(signal.answer);
                            break;

                        case 'ice-candidate':
                            await pc.addIceCandidate(signal.candidate);
                            break;
                    }
                } catch (error) {
                    console.error('Error handling signal:', error);
                }
            }

            sendSignal(toPeerId, signal) {
                if (!this.isFirebaseConfigured) return;
                
                const signalRef = ref(database, `rooms/${this.roomCode}/signals/${toPeerId}/${this.userId}`);
                set(signalRef, {
                    ...signal,
                    timestamp: serverTimestamp(),
                    from: this.userId
                });
            }

            setupSignalingListeners() {
                if (!this.isFirebaseConfigured) return;

                const participantsRef = ref(database, `rooms/${this.roomCode}/participants`);
                const participantsListener = onValue(participantsRef, (snapshot) => {
                    const participants = snapshot.val() || {};
                    
                    Object.keys(participants).forEach(peerId => {
                        if (peerId !== this.userId && !this.participants.has(peerId)) {
                            this.participants.set(peerId, participants[peerId]);
                            this.handleNewPeer(peerId);
                        } else if (peerId !== this.userId) {
                            this.participants.set(peerId, participants[peerId]);
                        }
                    });

                    this.participants.forEach((_, peerId) => {
                        if (!participants[peerId]) {
                            this.handlePeerDisconnection(peerId);
                        }
                    });

                    this.updateParticipantsList();
                });

                const signalsRef = ref(database, `rooms/${this.roomCode}/signals/${this.userId}`);
                const signalsListener = onValue(signalsRef, (snapshot) => {
                    const signals = snapshot.val() || {};
                    Object.keys(signals).forEach(fromPeerId => {
                        this.handleSignal(fromPeerId, signals[fromPeerId]);
                        set(ref(database, `rooms/${this.roomCode}/signals/${this.userId}/${fromPeerId}`), null);
                    });
                });

                this.signalingListeners.set('participants', { ref: participantsRef, listener: participantsListener });
                this.signalingListeners.set('signals', { ref: signalsRef, listener: signalsListener });
            }

            handlePeerDisconnection(peerId) {
                console.log('Peer disconnected:', peerId);
                
                const pc = this.peerConnections.get(peerId);
                if (pc) {
                    pc.close();
                    this.peerConnections.delete(peerId);
                }

                const audioElement = document.getElementById(`audio-${peerId}`);
                if (audioElement) {
                    audioElement.remove();
                }

                this.participants.delete(peerId);
                this.updateParticipantsList();
            }

            async joinRoom() {
                const usernameInput = document.getElementById('username');
                const roomCodeInput = document.getElementById('roomCode');
                const joinButton = document.getElementById('joinButton');
                
                if (!usernameInput.value.trim()) {
                    alert('Please enter your name');
                    return;
                }
                
                joinButton.disabled = true;
                joinButton.textContent = 'Connecting...';
                
                try {
                    this.username = usernameInput.value.trim();
                    this.roomCode = roomCodeInput.value.trim() || this.generateRoomCode();
                    
                    document.getElementById('status').textContent = 'Connecting...';
                    document.getElementById('status').className = 'status connecting';
                    
                    await this.startAudio();
                    
                    if (this.isFirebaseConfigured) {
                        this.roomRef = ref(database, `rooms/${this.roomCode}/participants/${this.userId}`);
                        await set(this.roomRef, {
                            username: this.username,
                            muted: this.isMuted,
                            joinedAt: serverTimestamp()
                        });

                        onDisconnect(this.roomRef).remove();
                        this.setupSignalingListeners();
                    } else {
                        this.simulateDemoMode();
                    }
                    
                    document.getElementById('setupSection').classList.add('hidden');
                    document.getElementById('intercomSection').classList.remove('hidden');
                    document.getElementById('status').textContent = `Connected to Room: ${this.roomCode}`;
                    document.getElementById('status').className = 'status connected';
                    
                    this.isConnected = true;
                    this.participants.set(this.userId, { username: this.username, muted: this.isMuted });
                    this.updateParticipantsList();
                    
                } catch (error) {
                    console.error('Error joining room:', error);
                    alert('Failed to join room. Please try again.');
                    document.getElementById('status').textContent = 'Connection Failed';
                    document.getElementById('status').className = 'status disconnected';
                } finally {
                    joinButton.disabled = false;
                    joinButton.textContent = 'Join Room';
                }
            }

            simulateDemoMode() {
                setTimeout(() => {
                    if (this.isConnected) {
                        const demoUsers = ['Alice', 'Bob', 'Charlie'];
                        demoUsers.forEach((name, index) => {
                            const demoId = `demo-${index}`;
                            this.participants.set(demoId, {
                                username: name,
                                muted: Math.random() > 0.7
                            });
                        });
                        this.updateParticipantsList();
                    }
                }, 2000);
            }

            async toggleMute() {
                this.isMuted = !this.isMuted;
                const muteButton = document.getElementById('muteButton');
                const audioStatus = document.getElementById('audioStatus');
                
                if (this.isMuted) {
                    muteButton.className = 'mute-button muted';
                    muteButton.textContent = '🔇';
                    audioStatus.innerHTML = '<span style="color: #ef4444;">●</span> Muted';
                    
                    this.peerConnections.forEach(pc => {
                        const senders = pc.getSenders();
                        senders.forEach(sender => {
                            if (sender.track && sender.track.kind === 'audio') {
                                pc.removeTrack(sender);
                            }
                        });
                    });
                } else {
                    muteButton.className = 'mute-button unmuted';
                    muteButton.textContent = '🎤';
                    audioStatus.innerHTML = '<span class="live-indicator"></span>Live Audio';
                    
                    if (this.localStream) {
                        this.peerConnections.forEach(pc => {
                            this.localStream.getAudioTracks().forEach(track => {
                                pc.addTrack(track, this.localStream);
                            });
                        });
                    }
                }
                
                if (this.isFirebaseConfigured && this.roomRef) {
                    set(ref(database, `rooms/${this.roomCode}/participants/${this.userId}/muted`), this.isMuted);
                }
                
                const participant = this.participants.get(this.userId);
                if (participant) {
                    participant.muted = this.isMuted;
                    this.updateParticipantsList();
                }
            }

            updateParticipantsList() {
                const list = document.getElementById('participantsList');
                list.innerHTML = '';
                
                this.participants.forEach((data, id) => {
                    const participantDiv = document.createElement('div');
                    participantDiv.className = 'participant';
                    
                    const connectionQuality = this.getConnectionQuality(id);
                    const qualityClass = connectionQuality >= 80 ? 'good' : connectionQuality >= 50 ? 'poor' : 'bad';
                    
                    participantDiv.innerHTML = `
                        <div class="participant-info">
                            <span class="connection-indicator ${qualityClass}"></span>
                            <span>${data.username}${id === this.userId ? ' (You)' : ''}</span>
                        </div>
                        <div style="display: flex; gap: 8px;">
                            <div class="speaking-indicator ${!data.muted && Math.random() > 0.8 ? 'active' : ''}"></div>
                            <div class="muted-indicator ${data.muted ? 'active' : ''}"></div>
                        </div>
                    `;
                    list.appendChild(participantDiv);
                });
            }

            getConnectionQuality(peerId) {
                const pc = this.peerConnections.get(peerId);
                if (!pc) return 100;
                return Math.floor(Math.random() * 40) + 60;
            }

            leaveRoom() {
                this.peerConnections.forEach(pc => pc.close());
                this.peerConnections.clear();
                
                document.querySelectorAll('audio[id^="audio-"]').forEach(audio => audio.remove());
                
                this.signalingListeners.forEach(({ ref: dbRef, listener }) => {
                    off(dbRef, 'value', listener);
                });
                this.signalingListeners.clear();
                
                if (this.isFirebaseConfigured && this.roomRef) {
                    set(this.roomRef, null);
                }
                
                if (this.localStream) {
                    this.localStream.getTracks().forEach(track => track.stop());
                    this.localStream = null;
                }
                
                if (this.volumeAnimation) {
                    cancelAnimationFrame(this.volumeAnimation);
                    this.volumeAnimation = null;
                }
                
                if (this.audioContext) {
                    this.audioContext.close();
                    this.audioContext = null;
                }
                
                this.isConnected = false;
                this.participants.clear();
                this.isMuted = false;
                
                document.getElementById('setupSection').classList.remove('hidden');
                document.getElementById('intercomSection').classList.add('hidden');
                document.getElementById('status').textContent = 'Disconnected';
                document.getElementById('status').className = 'status disconnected';
                
                const muteButton = document.getElementById('muteButton');
                muteButton.className = 'mute-button unmuted';
                muteButton.textContent = '🎤';
                
                document.getElementById('username').value = '';
                document.getElementById('roomCode').value = '';
            }
        }

        const intercom = new IntercomSystem();

        window.joinRoom = async function() {
            await intercom.joinRoom();
        };

        window.leaveRoom = function() {
            intercom.leaveRoom();
        };

        window.toggleMute = function() {
            intercom.toggleMute();
        };

        window.addEventListener('beforeunload', () => {
            intercom.leaveRoom();
        });
    </script>
</body>
</html>
