# Belt Clip Assistant Button — Setup Guide

One button. Hold to talk, release is enough (mic stays on). Short tap starts.
Long press (500ms) sends F14 which can be a separate Tasker action if you want it.

---

## What You Need

- Nice!Nano V2
- One momentary push button (any kind)
- Small LiPo battery (300-500mAh is plenty, lasts weeks with ZMK deep sleep)
- A GitHub account (free) to build the firmware

---

## Step 1 — Wire the Button

Wire one leg of the button to **D2 (pin P0.31)** on the Nice!Nano.
Wire the other leg to **GND**.

That's it. ZMK handles the internal pullup resistor, no extra components needed.

If you want to use a different pin, change the `31` in `beltclip.overlay` to match.
Nice!Nano pinout: https://nicekeyboards.com/docs/nice-nano/pinout-schematic

---

## Step 2 — Build the Firmware (GitHub Actions, no local tools needed)

1. Go to https://github.com/zmkfirmware/zmk-config-template
2. Click **Use this template** → **Create a new repository** → name it `zmk-beltclip`
3. In your new repo, replace the contents of `config/` with the files in this folder
4. Go to the **Actions** tab in your repo → the build starts automatically
5. When it finishes, download the artifact — it contains `beltclip-nice_nano_v2-zmk.uf2`

---

## Step 3 — Flash the Nice!Nano

1. Double-tap the reset button on the Nice!Nano — it shows up as a USB drive called `NICENANO`
2. Drag the `.uf2` file onto that drive
3. It reboots automatically, you're done

---

## Step 4 — Android: Smart Lock (Glove Mode)

This makes the phone stay unlocked while the belt clip is nearby.

1. Pair the belt clip to your phone via Bluetooth first (it shows up as "BeltClip")
2. Settings → Security → Smart Lock → Trusted Devices → Add trusted device → select BeltClip
3. Now when BeltClip is connected, phone stays unlocked. Screen still sleeps but no fingerprint needed.

---

## Step 5 — Install Tasker

Tasker is on the Play Store, costs a few dollars, worth it.

Create two tasks:

**Task 1: MicStart**
- Action: Launch App URL
- URL: `http://100.98.73.25:8000?mic=1`

**Task 2 (optional): MicStop**
- Action: Launch App URL  
- URL: `http://100.98.73.25:8000?stop=1`

Create two profiles:

**Profile 1:**
- Trigger: Hardware Key → F13 pressed
- Task: MicStart

**Profile 2 (optional):**
- Trigger: Hardware Key → F14 pressed
- Task: MicStop

---

## Step 6 — Web App Update

Add these four lines to the assistant's JavaScript, right after `let mediaRecorder=null...`:

```javascript
// Auto-start mic if URL has ?mic=1
const _urlParams = new URLSearchParams(window.location.search);
if (_urlParams.get('mic') === '1') {
    window.addEventListener('load', () => setTimeout(toggleRecording, 600));
}
```

---

## How It Works

- Button tap → F13 → Tasker opens `http://100.98.73.25:8000?mic=1` → mic starts in 0.6s
- You yap into it
- Tap stop in the app (or long-press button → F14 → Tasker can tap stop programmatically)
- Transcribes, sends, assistant responds

At the junkyard with gloves: phone stays unlocked via Smart Lock while belt clip is paired.
Button wakes screen, Tasker fires, mic is recording before you finish pulling the glove off.

---

## Battery Life Estimate

ZMK deep sleep kicks in after ~30 seconds of inactivity.
With a 500mAh LiPo and one button, expect 2-4 weeks between charges.
Nice!Nano has onboard charging via USB-C.
