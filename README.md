import hashlib
import os
import json

# Path to store the hash database
HASH_DB = "file_hashes.json"

def calculate_hash(file_path):
    """Calculate SHA-256 hash of a file."""
    sha256_hash = hashlib.sha256()
    try:
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                sha256_hash.update(chunk)
        return sha256_hash.hexdigest()
    except FileNotFoundError:
        print(f"[ERROR] File not found: {file_path}")
        return None


def load_hash_database():
    """Load stored file hashes from JSON database."""
    if os.path.exists(HASH_DB):
        with open(HASH_DB, "r") as f:
            return json.load(f)
    return {}


def save_hash_database(hashes):
    """Save file hashes to JSON database."""
    with open(HASH_DB, "w") as f:
        json.dump(hashes, f, indent=4)


def monitor_directory(directory):
    """Monitor all files in a directory for changes."""
    stored_hashes = load_hash_database()
    current_hashes = {}

    print(f"\n🔍 Scanning directory: {directory}")

    for root, _, files in os.walk(directory):
        for filename in files:
            file_path = os.path.join(root, filename)
            file_hash = calculate_hash(file_path)
            if file_hash:
                current_hashes[file_path] = file_hash

                # Compare with previous hash
                if file_path in stored_hashes:
                    if stored_hashes[file_path] != file_hash:
                        print(f"[ALERT] File Modified: {file_path}")
                else:
                    print(f"[NEW] New file detected: {file_path}")

    # Check for deleted files
    for file_path in stored_hashes:
        if file_path not in current_hashes:
            print(f"[DELETED] File missing: {file_path}")

    # Save updated hash database
    save_hash_database(current_hashes)
    print("\n✅ Scan complete. Hash database updated.")


if __name__ == "__main__":
    directory_to_monitor = input("Enter directory to monitor: ")
    monitor_directory(directory_to_monitor)

