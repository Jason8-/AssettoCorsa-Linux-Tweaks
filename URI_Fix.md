### How to Fix Content Manager URI Links (acmanager://) on Linux / Steam Deck

By default, Steam on Linux refuses to pass custom URI links (like NoHesi or SRP server invites) to Windows applications if they are already running inside a Proton prefix. Clicking a link will just bring Steam to the foreground.

To fix this, we can create a custom script that uses `protontricks` to bypass Steam and inject the link directly into the running game prefix. 

**Prerequisites:**
* Assetto Corsa installed with Content Manager renamed to `AssettoCorsa.exe`.
* Protontricks installed (e.g., `sudo pacman -S protontricks` or via Flatpak).

#### Step 1: Create the Auto-Detect Wrapper Script
This script automatically scans your Steam libraries to find your Assetto Corsa folder, then passes the link into Proton.

1. Open your terminal and create the script:
   ```bash
   nano ~/.local/bin/acmanager-live.sh
   ```
2. Paste the following code exactly as is:
   ```bash
   #!/bin/bash
   URI="$1"
   
   # 1. Locate Steam's master library config file
   VDF_FILE="$HOME/.steam/steam/steamapps/libraryfolders.vdf"
   if [ ! -f "$VDF_FILE" ]; then
       VDF_FILE="$HOME/.local/share/Steam/steamapps/libraryfolders.vdf"
   fi
   
   # 2. Extract library paths and find Assetto Corsa
   AC_DIR=""
   while read -r lib_path; do
       CHECK_DIR="$lib_path/steamapps/common/assettocorsa"
       if [ -d "$CHECK_DIR" ]; then
           AC_DIR="$CHECK_DIR"
           break
       fi
   done < <(grep -oP '"path"\s+"\K[^"]+' "$VDF_FILE")
   
   # 3. Change to the directory and inject the link
   if [ -n "$AC_DIR" ]; then
       cd "$AC_DIR" || exit
       protontricks -c "wine AssettoCorsa.exe \"$URI\"" 244210
   else
       echo "Error: Could not auto-detect Assetto Corsa installation folder."
       exit 1
   fi
   ```
3. Save and exit (In nano: **Ctrl+O**, **Enter**, **Ctrl+X**).
4. Make the script executable:
   ```bash
   chmod +x ~/.local/bin/acmanager-live.sh
   ```

#### Step 2: Create the Desktop Entry
We need to tell Linux to use this script whenever an `acmanager://` link is clicked.

1. Create a custom desktop file:
   ```bash
   nano ~/.local/share/applications/acmanager.desktop
   ```
2. Paste the following configuration (replace `yourusername` with your actual Linux username):
   ```ini
   [Desktop Entry]
   Name=Content Manager
   Exec=/home/yourusername/.local/bin/acmanager-live.sh "%u"
   Icon=steam_icon_244210
   Type=Application
   Terminal=false
   MimeType=x-scheme-handler/acmanager;
   NoDisplay=true
   ```
3. Save and exit.

#### Step 3: Register the MIME Type
Finally, set this new file as the default handler for the links and refresh your system's application database:

```bash
xdg-mime default acmanager.desktop x-scheme-handler/acmanager
update-desktop-database ~/.local/share/applications/
```

**That's it.** You can now click "Join" on any server page (or install links for mods), and it will immediately pop up in Content Manager, even if the game is already running.
