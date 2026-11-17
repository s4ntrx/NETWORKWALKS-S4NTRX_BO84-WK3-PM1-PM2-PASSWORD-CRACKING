# README: PDF Password Recovery Workflow (JTR, Johnny, OnlineHashCrack & Networkwalks)

This guide documents a controlled, offline password-recovery workflow for password-protected PDF files. It is intended **strictly for authorized security auditing, forensic investigation, or recovery of your own documents**. Unauthorized access to protected files is illegal.

---

## Prerequisites & Ethical Notice

* **Authorization**: Only proceed if you own the PDF or have explicit written permission from the owner to test its password strength.
* **Isolation**: Perform all cracking on a local machine or a controlled lab environment (e.g., Kali Linux VM or Windows with JTR Jumbo).
* **Tools Overview**:
  * **John the Ripper (Jumbo)**: The core cracking engine.
  * **Johnny**: A graphical front-end for John the Ripper (useful for Windows users).
  * **OnlineHashCrack**: A cloud-based GPU cracking service (requires upload of the file or hash).
  * **Networkwalks Tools**: Browser-based hash extraction and dictionary attack simulator.

---

## Method 1: Local Cracking with John the Ripper (CLI)

This is the most common method for security professionals using Linux (Kali/Parrot) or macOS.

### Step 1: Extract the Hash from the PDF

You cannot feed a PDF directly to John. You must use `pdf2john` to extract the crackable `$pdf$` hash.

**Linux / macOS / WSL:**
```bash
# Locate the script (usually in /usr/share/john/ or the JTR run directory)
python3 /usr/share/john/pdf2john.py /path/to/locked_document.pdf > pdf_hash.txt
```
**Windows (JTR Jumbo folder):**
```cmd
python pdf2john.py locked_document.pdf > pdf_hash.txt
```
*Note: The output file should contain a string starting with `$pdf$`.*

### Step 2: Run John the Ripper

Use a wordlist (e.g., `rockyou.txt`) to attempt the crack.
```bash
# Basic dictionary attack
john --wordlist=/usr/share/wordlists/rockyou.txt pdf_hash.txt

# If you want to use rules to mutate the words (recommended for better coverage)
john --wordlist=/usr/share/wordlists/rockyou.txt --rules pdf_hash.txt
```
*John will display the cracked password on the screen if found.*

### Step 3: Display the Recovered Password

If you missed the output or want to verify the result later:
```bash
john --show pdf_hash.txt
```

---

## Method 2: Local Cracking with Johnny (GUI for Windows)

Johnny is a graphical interface that wraps John the Ripper, making it easier for users who prefer not to use the command line.

### Step 1: Setup Johnny

1. Download and extract **John the Ripper Jumbo** for Windows from Openwall.
2. Download **Johnny** from the Openwall wiki.
3. Launch Johnny and go to **Settings**. Point the "John executable" path to `john.exe` (located in the `run` folder of the JTR Jumbo extraction).

### Step 2: Prepare the Hash File

Since Johnny requires a hash file, you still need to extract it.

* **Option A**: Use the `pdf2john.py` script included in the JTR folder via command prompt to create `hash.txt`.
* **Option B**: Use the **OnlineHashCrack PDF Hash Extractor** tool in your browser to generate the `$pdf$` string, then paste it into Notepad and save as `hash.txt`.

### Step 3: Crack in Johnny

1. Open Johnny and click **Open Password File**. Select your `hash.txt`.
2. Ensure the target is detected (it should show the format as `PDF`).
3. Click **Start New Attack**.
4. Johnny will run John in the background. Monitor the **Passwords** panel. Once the status shows "Cracked," the plaintext password will be displayed.

---

## Method 3: Cloud-Based Cracking (OnlineHashCrack)

If you lack local GPU power or need to crack a strong password quickly, you can use a paid cloud service.

### Step 1: Upload the File (or Hash)

1. Go to **onlinehashcrack.com**.
2. Upload the locked PDF directly. The service will automatically extract the hash and detect the encryption mode (e.g., RC4 or AES).
3. *Alternative (Forensic Best Practice)*: Extract the hash locally using `pdf2john` and upload only the text hash to keep the document content confidential.

### Step 2: Select Attack Type & Wait

* The service will automatically select a strategy (Dictionary, Brute-force, or Hybrid) based on the PDF's encryption level.
* **Basic Search** is free (tells you if the password is in their common database). Deeper searches (GPU hours) require payment.
* You will receive an email notification when the job completes. If found, you can view the password in your dashboard.

---

## Method 4: Browser-Based (Networkwalks Tools)

This method is purely for educational simulations or very weak passwords using built-in lists.

### Step 1: Extract Hash

1. Open the **Networkwalks Hash Calculator** (`networkwalks.com/hash-calculator/`).
2. Upload your locked PDF. The tool runs locally in your browser to generate the `$pdf$` hash.

### Step 2: Crack with Dictionary Attack

1. Copy the full hash (starting from `$pdf$`).
2. Open the **Networkwalks Password Cracker** (`networkwalks.com/password-cracker/`).
3. Paste the hash into the input box.
4. Run the attack using the **Built-in list (100 passwords)** or upload your own wordlist (.txt). This simulates the logic of John the Ripper but with limited computational power.

---

## Verification & Cleanup

1. **Verify**: Always try to open the original PDF with the recovered password to confirm it works. Do not rely solely on the tool's output.
2. **Cleanup**: If you extracted hashes, delete the hash files (`pdf_hash.txt`) after use, as they are sensitive artifacts. If you used a cloud service, ensure you understand their data retention policy (OnlineHashCrack states files are deleted after extraction, but hashes are kept).

## Troubleshooting

* **"No password hashes loaded"**: Ensure the hash file actually contains the `$pdf$` string and you saved it as a plain text file (UTF-8).
* **Crack is too slow**: PDF encryption (especially AES-256) is intentionally slow. For AES-256, dictionary attacks are usually the only feasible method; brute-force is generally impractical.
* **Johnny doesn't detect John**: Re-check the path in Settings. It must point directly to the `john.exe` binary, not the folder.
