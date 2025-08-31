# 🚀 Getting Started with the NRF51 Dev-Board

## 🛠️ Burning the CH32v203 DapLink Firmware

### 📁 Step 1: Get the DAPLink Firmware

1. Download the latest firmware release from the GitHub repository:
   👉 [DAPLink by XIVN1987](https://github.com/XIVN1987/DAPLink)

2. Extract the downloaded `.zip` file.

3. Navigate to:

   ```
   CH32V203 / obj / DAPLink.hex
   ```

   This `.hex` file will be used in the next step.



### ☄️ Step 2: Flash the Firmware

The onboard **CH32V203 MCU** can be programmed directly via USB using **WCHISPTool** from WCH — no external programmer required.

1. **Prepare the board:**
   - Short the jumper labeled `CH32V_Upload` with a conductive wire.
   - Then connect the board to your computer via USB.
   
2. **Open WCHISPTool:**
   
   - Select **Chip Family:** `CH32V20x`
   - Select **Target Chip:** `CH32V203F6P6`
   
3. **Load firmware:**
   - Ensure the interface is set to **USB**.
   
   - Choose the `DAPLink.hex` file from Step 1.
   
     <img src="https://github.com/TomatoCube18/NRF51_DevelopmentBoard/blob/main/images/WCHISPTool_Config.png"  width="500" height="auto" />
   
4. **Flash it:**
   
   - Click **Download** to write the firmware.
   - Once completed, remove the jumper to return the board to normal mode.







### 🔍 Step 3: Verify with pyOCD

After flashing, the board should enumerate as a **DAPLink USB device**. To confirm it works, you can use [pyOCD](https://github.com/pyocd/pyOCD).

1. Install pyOCD (requires Python 3.7+):

   ```
   pip install pyocd
   ```

2. Check if the board is detected:

   ```
   pyocd list
   ```

   You should see your NRF51822 board listed.

3. (Optional) Run a simple test to confirm communication:

   ```
   pyocd flash --target nrf51 --erase
   ```

   This ensures pyOCD can talk to the board and perform operations.

✅ If the board shows up in `pyocd list`, congratulations — your DAPLink setup is working!


<br>
<br>

## 🛠️ (A) Arduino Guide

This guide shows how to set up the **Arduino IDE** for the NRF51822 MCU, using the [`arduino-nRF5`](https://github.com/sandeepmistry/arduino-nRF5) core, and upload your first sketch to blink the onboard LED (P0.20).



### 🛠 Step 1: Install Arduino IDE

1. Download and install the latest Arduino IDE from:
   👉 [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)



### 📦 Step 2: Add the nRF5 Board Package

1. Open Arduino IDE.

2. Go to **File > Preferences**.

3. In the **Additional Board Manager URLs** field, add:

   ```
   https://sandeepmistry.github.io/arduino-nRF5/package_nRF5_boards_index.json
   ```

4. Click **OK** to save.



### 🧩 Step 3: Install the Board Support

1. Go to **Tools > Board > Boards Manager**.
2. Search for **nRF5 by Sandeep Mistry**.
3. Click **Install**.



### ⚙️ Step 4: Select the Board & Programmer

1. Go to **Tools > Board > nRF5 Boards > BBC micro:bit** (this uses the nRF51822 just like your board).

2. Under **Programmer**, select:

   ```
   CMSIS-DAP (SWD)
   ```



### 💡 Step 5: Blink LED on P0.20

1. Open **File > Examples > 01.Basics > Blink**.
2. Replace the pin number with **12** (which corresponds to P0.20):

```c
//
// Refer to MicroBit Pin_Naming from the NRF51 BSP 
// https://github.com/sandeepmistry/arduino-nRF5/blob/master/variants/BBCmicrobit/variant.cpp
//

const int LED_20 = 12;	// Set Pin_12(0.20) as output

// Blink example for NRF51822 custom board
void setup() {
  pinMode(LED_20, OUTPUT);   // Set P0.20 as output
}

void loop() {
  digitalWrite(LED_20, HIGH); // LED ON
  delay(1000);            // Wait 1 second
  digitalWrite(LED_20, LOW);  // LED OFF
  delay(1000);            // Wait 1 second
}
```

1. Click **Upload**.
   - The sketch will compile and flash via your DAPLink interface.
2. The LED connected to **12 / P0.20** should start blinking once per second.



### 💡 Step 5 (Extra): Blink LED on 5x5 Matrix

1. Open **File > Examples > 01.Basics > Blink**.
2. Replace the pin number with 1st Column  **3** (which corresponds to P0.03) & 1st row  **26** (which corresponds to P0.26):

```c
//
// Refer to MicroBit Pin_Naming from the NRF51 BSP 
// https://github.com/sandeepmistry/arduino-nRF5/blob/master/variants/BBCmicrobit/variant.cpp
//

const int COL1 = 3;
const int LED1 = 26;

void setup() {
  // Because the LED are multiplexed, we must ground the cathod side of the LED
  pinMode(COL1, OUTPUT);
  digitalWrite(COL1, LOW);
  
  pinMode(LED1, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED1, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED1, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
```

1. Click **Upload**.
   - The sketch will compile and flash via your DAPLink interface.
2. The Top Left LED on the 5x5 matrix connected to **3 / P0.04** & **26 / P0.13** should start blinking once per second.

<br>
<br>

## 🛠️ (B) MakeCode Arcade Electron Wrapper

This Node.js console app will:

- Take a universal `.hex` file as input
- Split it into `v1` and `v2` hex files
- Save them using the **original filename** as prefix (e.g., `myproject-v1.hex`, `myproject-v2.hex`)
- Using PyOCD, program the `v1` hex file to the target board.

------

### ✅ Prerequisites

- Node.js installed
- npm installed
- Terminal or command prompt
- PyOCD Installed

------

### 📁 Step 1: Create your project folder

```bash
mkdir nrf51-makecode-browser
cd nrf51-makecode-browser
```

------

### 📦 Step 2: Initialize the project

```bash
npm init -y
```

------

### 📚 Step 3: Install the Microbit Universal Hex library & Electron Library

```bash
npm install @microbit/microbit-universal-hex
npm install --save-dev electron

```

------

### 📝 Step 4: Add npm start command to `package.json` file

Edit `package.json` by adding the following line within the scripts section.

`"start": "electron index.js`

Example:

```json
{
  "name": "nrf51-makecode-browser",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "electron index.js"   // add the line here
  },
  ...
}
    
```

------

### 📝 Step 5: Create the `index.js` script

```bash
touch index.js
```

Edit `index.js` and add the following code:

```javascript
// main.js
const { app, BrowserWindow, session, ipcMain, shell } = require('electron');
const { execFile } = require('child_process');

const path = require('path');
const fs = require('fs');
const { separateUniversalHex, microbitBoardId } = require('@microbit/microbit-universal-hex');

// ▼ Download folder lives next to this script:
const DOWNLOAD_DIR = path.join(__dirname, 'downloads');

// Check if download folder exists
if (!fs.existsSync(DOWNLOAD_DIR)) {
  // If not, create it (including any parent folders)
  fs.mkdirSync(DOWNLOAD_DIR, { recursive: true });
  console.log(`Created folder: ${DOWNLOAD_DIR}`);
} else {
  console.log(`Folder already exists: ${DOWNLOAD_DIR}`);
}


function createWindow() {
  // ensure the download folder exists
  if (!fs.existsSync(DOWNLOAD_DIR)) {
    fs.mkdirSync(DOWNLOAD_DIR, { recursive: true });
  }

  const win = new BrowserWindow({
    width: 1280,
    height: 800,
    webPreferences: {
      nativeWindowOpen: true,

      nodeIntegration: true,    // So ipcRenderer is available   //false,
      contextIsolation: false, //true,
      webviewTag: true           // enable <webview> support
    },
  });

  // Intercept every download…
  session.defaultSession.on('will-download', (event, item) => {
    const filename = item.getFilename();
    const savePath = path.join(DOWNLOAD_DIR, filename);
    item.setSavePath(savePath);

    // When download finishes:
    item.once('done', (e, state) => {
      if (state === 'completed') {
        console.log(`✅ Downloaded: ${savePath}`);
        // If it’s a .hex, flash via pyOCD
          const escaped = savePath.replace(/ /g, '\\ ');

          const inputFilePath = savePath;
          const baseName = path.basename(inputFilePath, path.extname(inputFilePath)); // e.g., myproject

          fs.readFile(inputFilePath, 'utf8', (err, data) => {
              if (err) {
                  console.error(`❌ Error reading file "${inputFilePath}":`, err.message);
                  process.exit(1);
              }

              try {
                  const separatedBinaries = separateUniversalHex(data);

                  let v1Found = false;
                  let v2Found = false;

                  separatedBinaries.forEach(hexObj => {
                      if (hexObj.boardId === microbitBoardId.V1) {
                          path.join(__dirname, 'downloads');
                          const v1Path = path.join('downloads', `${baseName}-v1.hex`);
                          // const v1Path = `${baseName}-v1.hex`;
                          fs.writeFileSync(v1Path, hexObj.hex, 'utf8');
                          console.log(`✅ ${v1Path} created`);

                          v1Found = true;
                      } else if (hexObj.boardId === microbitBoardId.V2) {
                          const v2Path = path.join('downloads', `${baseName}-v2.hex`);
                          // const v2Path = `${baseName}-v2.hex`;
                          fs.writeFileSync(v2Path, hexObj.hex, 'utf8');
                          console.log(`✅ ${v2Path} created`);
                          v2Found = true;
                      }
                  });

                  if (!v1Found) {
                      console.warn('⚠️ No v1 hex found in universal hex');
                  }

                  if (!v2Found) {
                      console.warn('⚠️ No v2 hex found in universal hex');
                  }

              } catch (e) {
                  console.error('❌ Failed to parse universal hex file:', e.message);
                  process.exit(1);
              }
          });

          


          // Buffer to accumulate all text from “pyocd list”
          let listOutput = '';

          // Send a quick header message to the log-pane
          win.webContents.send('pyocd-stdout', '\n>>> Checking for connected debug probes…\n');

          // 1) STEP 1: spawn “pyocd list”
          const listProc = execFile('pyocd', ['list'], { shell: true });

          listProc.stdout.on('data', (chunk) => {
            const text = chunk.toString();
            listOutput += text;
            win.webContents.send('pyocd-stdout', text);
          });

          listProc.stderr.on('data', (chunk) => {
            const text = chunk.toString();
            listOutput += text;
            win.webContents.send('pyocd-stderr', text);
          });

          listProc.on('close', (code) => {
            // After “pyocd list” completes, decide if we proceed or abort
            const trimmed = listOutput.trim();

            if (
              trimmed === '' ||
              listOutput.includes('No available debug probes are connected')
            ) {
              // No probes found → abort
              win.webContents.send(
                'pyocd-stdout',
                '\n>>> ERROR: No debug probes detected. Aborting erase/flash.\n'
              );
              win.webContents.send('pyocd-done');
              return;
            }

            // If we reach here, at least one probe was listed.
            win.webContents.send(
              'pyocd-stdout',
              '\n>>> Probe(s) detected. Proceeding to erase and flash.\n'
            );


          const erase = execFile('pyocd', ['erase', '-t', 'nrf51822', '--chip'], { shell: true });


          // Forward stdout chunks
          erase.stdout.on('data', (chunk) => {
            win.webContents.send('pyocd-stdout', chunk.toString());
          });
          // Forward stderr chunks
          erase.stderr.on('data', (chunk) => {
            win.webContents.send('pyocd-stderr', chunk.toString());
          });

          erase.on('close', (code) => {
            if (code !== 0) {
              win.webContents.send('pyocd-stdout', `\n>>> Erase exited with code ${code}\n`);
              win.webContents.send('pyocd-done');
              return;
            }

            const v1Path = path.join('downloads', `${baseName}-v1.hex`);

            const flash = execFile('pyocd', ['flash', '-t', 'nrf51822', v1Path], { shell: true });
            
            flash.stdout.on('data', (chunk) => {
              win.webContents.send('pyocd-stdout', chunk.toString());
            });
            flash.stderr.on('data', (chunk) => {
              win.webContents.send('pyocd-stderr', chunk.toString());
            });
            flash.on('close', (flashCode) => {
              win.webContents.send('pyocd-stdout', `\n>>> Flash exited with code ${flashCode}`);
              win.webContents.send('pyocd-done');
            });
          });
        

        });


      } else {
        console.error(`Download failed: ${state}`);
      }
    });
  });

  // Load any URL:
  //win.loadURL('https://makecode.microbit.org');
  win.loadFile('log.html');
}

app.whenReady().then(createWindow);


ipcMain.on('request-open-download-folder', () => {
  console.log('⏳ Download Button clicked - IPCMain');
  const DOWNLOAD_DIR = path.join(__dirname, 'downloads');
  shell.openPath(DOWNLOAD_DIR).then(errMsg => {
    if (errMsg) {
      console.error('Shell.openPath failed:', errMsg);
    }
  });
});

ipcMain.on('request-open-external', () => {
  const extWin = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: { nodeIntegration: false, contextIsolation: true }
  });
  extWin.loadURL('https://tomatocube.com/wiki/TC-CX-PY-101-00/PSC_Python_PCEP_Pilot_2025.html#pep8-exercise');
});

app.on('window-all-closed', () => {
  app.quit();
});

// app.on('window-all-closed', () => {
//   if (process.platform !== 'darwin') app.quit();
// });
```



### ✅ Step 6: Create the `log.html` script

```bash
touch log.html
```

Edit `log.html` and add the following code:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>.:: TomatoCube* NRF51 MakeCode Wrapper ::.</title>
  <style>
    /* Make the body fill the entire window, split into two rows */
    html, body {
      margin: 0;
      padding: 0;
      height: 100vh;
      display: flex;
      flex-direction: column;
      background: #1e1e1e;
      color: #e5e5e5;
      font-family: monospace;
    }

    /* Top section: the <webview> hosting MakeCode */
    #mainView {
      flex: 1;             /* Fill all available vertical space */
      border: none;        /* Remove default border */
      min-height: 0;       /* Prevent overflow issues in flex containers */
    }

    /* Bottom section: the live pyOCD log */
    #logContainer {
      flex: 0 0 15%;       /* Reserve ~15% of height for the log */
      background: #1e1e1e;
      border-top: 1px solid #444;
      overflow-y: auto;    /* Scroll if log grows too large */
    }

    /* <pre> inside log container */
    #log {
      margin: 0;
      padding: 0.5rem;
      white-space: pre-wrap;   /* Wrap long lines */
      word-break: break-word;
    }


    /* Floating button: bottom-right corner */
    #openPageBtn {
      position: fixed;
      bottom: 20px;
      right: 20px;
      z-index: 1000;               /* Always on top */
      padding: 0.6rem 1rem;
      background-color: #007ACC;   /* Example color */
      color: white;
      border: none;
      border-radius: 4px;
      font-size: 1rem;
      cursor: pointer;
      box-shadow: 0 2px 6px rgba(0,0,0,0.3);
    }

    #openPageBtn:hover {
      background-color: #005EA6;
    }

    /* New “Open Folder” button, just above the other one */
    #openFolderBtn {
      position: fixed;
      bottom: 70px;       /* push it up by ~50px so they don’t overlap */
      right: 20px;
      z-index: 999;
      padding: 0.6rem 1rem;
      background-color: #28A745;  /* greenish accent */
      color: white;
      border: none;
      border-radius: 4px;
      font-size: 1rem;
      cursor: pointer;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
    }
    #openFolderBtn:hover {
      background-color: #218838;
    }

  </style>
</head>
<body>
  <!-- Top half: use <webview> to load MakeCode (avoids X-Frame-Options issues) -->
  <webview
    id="mainView"
    src="https://makecode.microbit.org"
    preload=""
    style="background: #fff;"
  ></webview>

  <!-- Bottom half: live pyOCD logs -->
  <div id="logContainer">
    <pre id="log">Waiting for pyOCD output…</pre>
  </div>

  <script>
    // Assume nodeIntegration: true or a preload that exposes ipcRenderer
    const { ipcRenderer } = require('electron');

    // Grab references to the <pre> and its container
    const logContainer = document.getElementById('logContainer');
    const logEl        = document.getElementById('log');

    // Helper to append text and then auto-scroll
    function appendToLog(text, isError = false) {
      if (isError) {
        // Prefix stderr lines if desired
        logEl.textContent += '\n[stderr] ' + text;
      } else {
        logEl.textContent += text;
      }
      // Force scroll to bottom
      logContainer.scrollTop = logContainer.scrollHeight;
    }

    // Listen for stdout chunks
    ipcRenderer.on('pyocd-stdout', (_, chunk) => {
      appendToLog(chunk, false);
    });

    // Listen for stderr chunks
    ipcRenderer.on('pyocd-stderr', (_, chunk) => {
      appendToLog(chunk, true);
    });

    // When the process finishes
    ipcRenderer.on('pyocd-done', () => {
      appendToLog('\n\n>>> pyOCD process complete.', false);
    });
  </script>


  <!-- Floating “Open Page” button -->
  <button id="openPageBtn">Open Tutorial Page</button>

  <!-- Button #2: Open “downloads” folder in OS file explorer -->
  <button id="openFolderBtn">Show Downloads Folder</button>


  <script>
     
    document.getElementById('openPageBtn').addEventListener('click', () => {
      console.log('Renderer: “Open MakeCode” clicked');
      ipcRenderer.send('request-open-external');
    });

    document.getElementById('openFolderBtn').addEventListener('click', () => {
      console.log('Renderer: “Show Downloads Folder” clicked');
      ipcRenderer.send('request-open-download-folder');
    });

   
  </script>


</body>
</html>

```



### Folder Structure Cheking

Make sure you have the following structure up to this point. The subfolder called `downloads` will be created if it does not exist :

```tex
/your‐project
  ├─ index.js
  ├─ log.html        
  ├─ /downloads      ←—— this folder will be created at runtime, if it doesn't exist.
  │     project1.hex
  │     project1-v1.hex
  │     project1-v2.hex
  │     …
  └─ package.json
```



### ✅ Step 7: Run the Electron script

```bash
npm start
```

- Your Electron window opens.
- Any file you download lands in `./downloads` under your project.
- If a `.hex` is downloaded, the file will first be split into `-v1` & `-v2` for microbit v1 (nrf51) & Microbit v2 (nrf52) accordingly. 
- After splitting is completed, pyOCD will be used to `erase` then `flash` the hex file through a debug probe to your target board. It's console output will be presented in your log Window.

------

You’re all set—every download now saves to a fixed `downloads/` folder, and `.hex` files get auto-flashed via pyOCD!

<br>
<br>

## 🐍 (C) Getting Started with MicroPython

This guide shows how to flash **MicroPython (micro:bit build)** and blink an LED on **P0.20**, using **pyOCD** and a simple serial terminal (e.g., Arduino Serial Monitor at **115200 baud**).



### 📁 Step 1: Get the MicroPython Firmware

1. Download the latest micro:bit MicroPython `.hex` from:
   👉 [https://micropython.org/download/MICROBIT/](https://micropython.org/download/MICROBIT/)
2. Note the filename (e.g., `MICROBIT-20250415-v1.25.0.hex`).



### ⚡ Step 2: Flash with pyOCD

> Requires Python + `pyocd` (`pip install pyocd`)

```
pyocd erase -t nrf51822 --chip
pyocd flash -t nrf51822 MICROBIT-20250415-v1.25.0.hex
```



### 🖥️ Step 3: Open the REPL

1. Open **Arduino IDE → Tools → Serial Monitor** (or your favorite terminal).
2. Set **Baud: 115200**, **Both NL & CR** (if available).
3. You should see the MicroPython prompt: `>>>`



### 💡 Step 4: Blink LED on P0.20

### Option A — Using standard `machine` API (if available in your build)

```python
import machine, time
led = machine.Pin(20, machine.Pin.OUT)

for i in range(10):
    led.value(1)   # ON
    time.sleep(1)
    led.value(0)   # OFF
    time.sleep(1)
```

### Option B — Using micro:bit API (if your firmware exposes `microbit`)

```python
from microbit import *
# On some boards, P0.20 may map to 'pin20'
for i in range(10):
    pin20.write_digital(1)
    sleep(1000)
    pin20.write_digital(0)
    sleep(1000)
```
