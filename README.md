# I Built My Own Mini Spotify Server on My Android Phone

This project turns a normal Android phone into a complete self-hosted music streaming system — a fully private “mini Spotify” built using Termux, Ubuntu (via proot-distro), Navidrome, and open-source tools.

The purpose behind this setup was straightforward:  
Build a zero-cost, privacy-controlled, ad-free streaming platform that I own completely. No subscriptions. No ads. No sending data to any company. All music is stored by me, streamed by me, and controlled by me.

---

## 🟦 Why I Built This

Typical streaming apps restrict how you use your own library.  
I wanted a system where:

- I own all the files  
- I decide who can access it  
- It works offline (hotspot-only)  
- It streams instantly across devices  
- The stack is built on real backend concepts  

What began as a learning experiment evolved into a reliable, personal streaming setup running inside a phone.

---

## 🟧 System Architecture

### **Android Layer**
- Physical file storage  
- Hotspot / Wi-Fi to serve clients  
- Termux as the Linux environment host  

### **Termux Layer**
- Runs a complete Ubuntu installation through proot-distro  
- Provides Linux environment and persistent filesystem  

### **Ubuntu (proot) Layer**
I created a single dedicated Linux user:

- **User: `spotirip`**  
  Responsible only for running the Navidrome server.  
  This user has no downloading rights, no external automation — only indexing and serving pre-existing local audio files.

### **Shared Storage**
A directory inside Android’s internal storage is mounted into Ubuntu.  
Navidrome reads the music from this folder and indexes it automatically.

### **Navidrome Layer**
Navidrome handles all streaming features:
- Modern web UI  
- Album art, playlists, metadata  
- Fast indexing and scanning  
- Subsonic API for mobile clients  
- Works locally or through secure tunnels  

### **Client Layer**
Any device on the hotspot/Wi-Fi can stream through:
- Browser  
- Subsonic-compatible apps (e.g., Ultrasonic)  
- Another phone or laptop  

The result is a clean, private music server similar to Spotify, but fully self-hosted.

---

## 🟨 Installation & Setup (All Commands Included)

This is the complete, safe, legally clean installation procedure.

### 1. Termux base setup

Update packages:

```sh
pkg update && pkg upgrade -y
```

Install proot-distro:

```sh
pkg install proot-distro -y
```

---

### 2. Install Ubuntu inside Termux

Install Ubuntu image:

```sh
proot-distro install ubuntu
```

Enter Ubuntu:

```sh
proot-distro login ubuntu
```

Inside Ubuntu, update packages:

```sh
apt update && apt upgrade -y
```

---

### 3. Create a dedicated service user for the music server

Inside Ubuntu:

```sh
adduser spotirip
```

You can switch to it using:

```sh
su - spotirip
```

---

### 4. Install required tools

From Ubuntu:

```sh
apt update
apt install wget unzip ffmpeg nano -y
```

(Ffmpeg is only for metadata reading — **not for downloading**.)

---

### 5. Download and install Navidrome

Inside Ubuntu:

```sh
cd /opt
wget https://github.com/navidrome/navidrome/releases/latest/download/navidrome_linux_arm64.zip
unzip navidrome_linux_arm64.zip
rm navidrome_linux_arm64.zip
```

Now Navidrome is located at:

```
/opt/navidrome/
```

---

### 6. Create the shared music directory

This is where you place **your own legally obtained audio files**.

```sh
mkdir -p /storage/emulated/0/music_server
```

You can copy files into this folder using your file manager.

---

### 7. Configure Navidrome

Inside Ubuntu:

```sh
nano /opt/navidrome/navidrome.toml
```

Insert:

```
MusicFolder = "/storage/emulated/0/music_server"
ScanInterval = "1m"
Address = "0.0.0.0"
Port = 4533
```

Save and exit with `CTRL+O`, `Enter`, `CTRL+X`.

---

### 8. Start Navidrome

Switch to the service user:

```sh
su - spotirip
```

Start the server:

```sh
cd /opt/navidrome
./navidrome
```

If successful, you will see:

```
Navidrome server is ready! address="0.0.0.0:4533"
```

---

### 9. Get your local IP address (for hotspot/Wi-Fi)

Open a **new Termux session** (outside Ubuntu):

```sh
ip -4 addr show wlan0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

Example output:

```
192.168.43.1
```

---

### 10. Connect from other devices

On any device connected to your phone’s network, open:

```
http://<your-ip>:4533
```

Example:

```
http://192.168.43.1:4533
```

You will see the Navidrome UI and can stream your music instantly.

Subsonic-compatible clients work as well.

---

## 🟩 Workflow Summary

1. Open Termux  
2. Start Ubuntu:

   ```sh
   proot-distro login ubuntu
   ```

3. Switch to `spotirip`:

   ```sh
   su - spotirip
   ```

4. Start Navidrome:

   ```sh
   cd /opt/navidrome
   ./navidrome
   ```

5. On your other device, open the server address.

6. To add new music, simply copy audio files into:

```
/storage/emulated/0/music_server
```

Navidrome automatically scans the folder.

---

## 🟫 Technical Highlights

- Self-hosted Linux on Android  
- Runs entirely without root  
- Clean and legally compliant (no download automation)  
- Separation of concerns using dedicated user (`spotirip`)  
- Local-first architecture using hotspot/Wi-Fi  
- Subsonic API support  
- Streaming server built on lightweight Golang backend  

A lightweight, private, personal cloud music system — all powered by a phone.

---

## 🟪 Future Enhancements

I am now working on making the server available securely from anywhere in the world.

Exploring:

1. Cloudflare Tunnels  
2. VPN routing (WireGuard / Tailscale)  
3. Optional remote storage mounts (self-owned only, legal content only)

The long-term goal:  
**A fully private, globally accessible music system** using my own infrastructure.

---

