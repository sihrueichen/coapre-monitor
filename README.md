# coapre-monitor
廠護報名課程-通知
import requests
import hashlib
import os

URLS = [
    'https://www.coapre.org.tw/tw/',
    'https://www.coapre.org.tw/tw/education-register'
]

HASH_FILE = 'hashes.txt'

LINE_TOKEN = os.getenv("LINE_NOTIFY_TOKEN")

def fetch_hash(url):
    r = requests.get(url)
    r.encoding = r.apparent_encoding
    return hashlib.sha256(r.text.encode('utf-8')).hexdigest()

def send_line_notify(msg):
    if not LINE_TOKEN:
        print("No LINE token set, skipping notify.")
        return
    requests.post(
        "https://notify-api.line.me/api/notify",
        headers={"Authorization": f"Bearer {LINE_TOKEN}"},
        data={"message": msg}
    )

def load_previous_hashes():
    if not os.path.exists(HASH_FILE):
        return {}
    with open(HASH_FILE, 'r') as f:
        return dict(line.strip().split(' ', 1) for line in f)

def save_hashes(hashes):
    with open(HASH_FILE, 'w') as f:
        for url, h in hashes.items():
            f.write(f"{url} {h}\n")

def main():
    prev_hashes = load_previous_hashes()
    curr_hashes = {}
    changes = []

    for url in URLS:
        h = fetch_hash(url)
        curr_hashes[url] = h
        if url in prev_hashes and prev_hashes[url] != h:
            changes.append(url)

    save_hashes(curr_hashes)

    if changes:
        msg = "[COAPRE] 網頁有更新：\n" + "\n".join(changes)
        send_line_notify(msg)
        print(msg)
    else:
        print("No changes.")

if __name__ == "__main__":
    main()
