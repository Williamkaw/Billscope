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

def run_single_url_scraper(url, output_file):
    if not url.startswith("http"):
        url = "https://" + url

    try:
        print(f"\n🌐 Fetching {url} ...")
        html = read_html(url)
        emails = extract_emails(html)

        if emails:
            print(f"✅ Found {len(emails)} emails from {url}:")
            for email in emails:
                print(f"   📧 {email}")
        else:
            print(f"⚠️ No emails found on {url}")

        os.makedirs(os.path.dirname(output_file), exist_ok=True)
        with open(output_file, 'a', newline='') as f:
            writer = csv.writer(f)
            for email in emails:
                writer.writerow([email, url])

        print(f"📁 Saved to {output_file}")
    except Exception as e:
        print(f"❌ Error fetching {url}: {e}")

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
    print("\n🔍 BillScope OSINT Toolkit")
    print("1. Extract emails from a URL")
    print("2. Scan usernames across social media")
    print("3. Exit")
    choice = input("\nEnter your choice: ").strip()

    if choice == '1':
        url = input("Enter a URL (e.g. https://example.com): ").strip()
        run_single_url_scraper(url, "results/emails.csv")
    elif choice == '2':
        username = input("Enter the username to search: ").strip()
        scan_username(username)
    else:
        print("👋 Exiting...")

if __name__ == "__main__":
    main()


