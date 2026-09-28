# Hand Tracking AI Mouse

A computer vision-driven virtual mouse and gesture-control system built in Python for macOS. Using real-time hand-landmark estimation via Google MediaPipe and OpenCV, this tool translates hand poses, finger joint extensions, and dynamic gestures into native OS-level cursor movements, mouse clicks, continuous scrolling, and multi-desktop space navigation.
## Demo

https://github.com/user-attachments/assets/8bd920ac-48b3-4912-bbea-56a095d89d97

Above: Real-time demonstration showcasing cursor translation, pinch clicks, posture-based scrolling, and the open-palm to fist workspace swipe gesture.*

## Features

* **Sub-Pixel Cursor Tracking**: Computes the centroid across key palm landmarks (IDs `0, 5, 9, 13, 17`) to ensure steady, reliable pointer tracking.
* **Dead-Zone Filtering**: Eliminates webcam jitter when holding the hand steady without sacrificing rapid cursor responsiveness.
* **Interpolated Sensitivity Window**: Maps a centrally bounded active tracking box (`FRAME_R`) to the full display resolution, minimizing required arm travel.
* **Click Controls (Pinch Dynamics)**:
  * **Left Click**: Pinch Index Finger + Thumb
  * **Right Click**: Pinch Middle Finger + Thumb
  * **Double Click**: Pinch Ring Finger + Thumb
* **Continuous Posture-Based Scroll**: Pinch Index and Middle fingers to engage scrolling mode; keeping fingers extended scrolls upward continuously, while curling them toward the knuckle scrolls downward.
* **macOS Space Switching (State-Machine Gesture)**:
  * Open palm to prime tracking (`PALM READY`).
  * Close fist to lock tracking (`FIST LOCKED`).
  * Drag horizontally across the threshold to trigger `Ctrl + Left` or `Ctrl + Right`, transitioning between macOS Mission Control desktops.

## Gesture Reference

| Gesture | Fingers / Pose | Action | Visual Indicator |
| :--- | :--- | :--- | :--- |
| **Cursor Move** | Open palm (steering via palm center) | Move pointer across screen | Hand Landmark Skeleton |
| **Left Click** | Index Tip (`8`) + Thumb Tip (`4`) | `pyautogui.click()` | Green Circle |
| **Right Click** | Middle Tip (`12`) + Thumb Tip (`4`) | `pyautogui.rightClick()` | Blue Circle |
| **Double Click** | Ring Tip (`16`) + Thumb Tip (`4`) | `pyautogui.doubleClick()` | Red Circle |
| **Scroll Up** | Index (`8`) + Middle (`12`) joined, straight | `pyautogui.scroll(1)` | Yellow Circle |
| **Scroll Down** | Index (`8`) + Middle (`12`) joined, folded | `pyautogui.scroll(-1)` | Yellow Circle |
| **Workspace Swipe** | Open Palm → Closed Fist → Horizontal Drag | Desktop Space Shift (`Ctrl + Arrow`) | On-Screen State Overlay |

## Architecture & Logic

```mermaid
flowchart TD
    WS["Webcam Stream (640x480)"] --> MP["MediaPipe Hands Pipeline (21 3D Landmarks)"]
    
    MP --> F_Det["Fist State Detected"]
    MP --> F_Inact["Fist State Inactive"]
    
    F_Det --> DDC["Drag Distance Calculation"]
    DDC --> LHR[("- Left / Right Threshold")]
    DDC --> TDH[("- Triggers Desktop Hotkeys")]
    
    F_Inact --> IMJ{"Index + Middle Joined?"}
    
    IMJ -- "Yes" --> KTD["Knuckle-Tip Delta"]
    KTD --> PGS["pyAutoGUI Scroll"]
    
    IMJ -- "No" --> PC["Palm Centroid"]
    PC --> NSM["Normalized Screen Map"]
    
    IMJ -- "No" --> PCh["Pinch Checks"]
    PCh --> LRD["Left / Right / Double Click"]
```
## Setup & Installation

### 1. Clone the Repository
```bash
git clone [https://github.com/bhagwatigb/Hand-Tracking-Mouse.git](https://github.com/bhagwatigb/Hand-Tracking-Mouse.git)
cd Hand-Tracking-Mouse
```



### 2. Configure Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install requirements.txt
```

### 4. Configure macOS permissions
macOS limits automated mouse and keyboard events by default:

* Open **System Settings > Privacy & Security > Accessibility**.
* Enable your terminal emulator (e.g., **Terminal**, **iTerm2**, or your IDE).
* Under **System Settings > Privacy & Security > Camera**, ensure access is granted.
* Verify Mission Control desktop switching is enabled under **System Settings > Keyboard > Keyboard Shortcuts > Mission Control** (`Move left a space` / `Move right a space`).

## Usage
Run the tracking loop from within your activated environment
```bash
pythoon main.py
```
* Exit Application: Focus on the OpenCV webcam window and press q.
* Failsafe: Slam the physical mouse pointer into any corner of the display to trip PyAutoGUI's safety abort.

## Technical Stack

* Language: Python 3.9+
* Computer Vision: OpenCV (cv2)
* ML Inference: MediaPipe Hands
* OS Automation: PyAutoGUI
* Numerical Processing: NumPy
