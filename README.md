# Billscope
#!/usr/bin/env python3
import re
import os
import time
import urllib.request
import subprocess
import csv

# ------------------ EMAIL SCRAPER FUNCTIONS ------------------ #

email_regex = re.compile(r'''
(
[a-zA-Z0-9_.+]+
@
[a-zA-Z0-9_.+]+
)
''', re.VERBOSE)

def extract_emails(text):
    matches = email_regex.findall(text)
    return list(set(matches))  # remove duplicates

def read_html(url):
    headers = {'User-Agent': 'Mozilla/5.0'}
    req = urllib.request.Request(url, headers=headers)
    with urllib.request.urlopen(req, timeout=10) as res:
        return res.read().decode()

def run_email_scraper(input_file, output_file):
    start_time = time.time()
    results = []

    if not os.path.exists(input_file):
        print(f"❌ Input file not found: {input_file}")
        return

    with open(input_file, 'r') as infile:
        urls = [line.strip() for line in infile.readlines() if line.strip()]

    for i, url in enumerate(urls, 1):
        try:
            html = read_html(url)
            emails = extract_emails(html)
            print(f"{i}. {url} → {len(emails)} emails")
            for email in emails:
                results.append((email, url))
        except Exception as e:
            print(f"{i}. {url} → Error: {e}")

    os.makedirs(os.path.dirname(output_file), exist_ok=True)
    with open(output_file, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['Email', 'Source URL'])
        writer.writerows(results)

    print(f"✅ Saved {len(results)} emails to {output_file}")
    print(f"⏱️ Time taken: {round(time.time() - start_time, 2)} seconds")

# ------------------ USERNAME SCANNER FUNCTIONS ------------------ #

def scan_username(username):
    output_dir = "results/sherlock_results"
    os.makedirs(output_dir, exist_ok=True)
    output_file = os.path.join(output_dir, f"{username}.txt")

    print(f"\n🔎 Scanning for username: {username}...\n")
    try:
        subprocess.run(["sherlock", username, "--output", output_file], check=True)
        print(f"\n✅ Scan complete. Results saved in {output_file}")
    except Exception as e:
        print(f"❌ Sherlock error: {e}")

# ------------------ MAIN MENU ------------------ #

def main():
    os.makedirs("results/sherlock_results", exist_ok=True)
    print("\n🔍 OSINT Toolkit")
    print("1. Extract emails from URLs")
    print("2. Scan usernames across social media")
    print("3. Exit")
    choice = input("\nEnter your choice: ").strip()

    if choice == '1':
        input_file = input("Enter path to URL list (e.g. urls.txt): ").strip()
        output_file = "results/emails.csv"
        run_email_scraper(input_file, output_file)
    elif choice == '2':
        username = input("Enter the username to search: ").strip()
        scan_username(username)
    else:
        print("👋 Exiting...")

if __name__ == "__main__":
    main()

