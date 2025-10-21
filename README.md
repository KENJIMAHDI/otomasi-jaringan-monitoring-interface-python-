# otomasi-jaringan-monitoring-interface-python-
# berisi terkait dengan monitoring interface pada python yang di jalankan pada devasc-lab

#!/usr/bin/env python3
import subprocess
import time
import os

def get_interfaces_status():
    """
    Mengambil daftar interface jaringan dan statusnya (UP atau DOWN)
    menggunakan perintah 'ip link show'.
    """
    result = subprocess.run(["ip", "link", "show"], capture_output=True, text=True)
    lines = result.stdout.split('\n')
    interfaces = {}

    for line in lines:
        if ": " in line:
            parts = line.split(": ")
            if len(parts) >= 2:
                name = parts[1].split()[0]
                # Mengecek status UP/DOWN
                if "state UP" in line:
                    interfaces[name] = "UP"
                elif "state DOWN" in line:
                    interfaces[name] = "DOWN"
    return interfaces


def disable_interface(interface):
    """
    Menonaktifkan interface yang statusnya DOWN menggunakan perintah ip.
    """
    print(f"[ACTION] Menonaktifkan interface: {interface}")
    subprocess.run(["sudo", "ip", "link", "set", interface, "down"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)


def show_interfaces(interfaces):
    """
    Menampilkan daftar interface dan statusnya.
    """
    print("\n=== STATUS INTERFACE ===")
    for name, status in interfaces.items():
        print(f" - {name:<10} : {status}")
    print("========================\n")


def main():
    # Pastikan dijalankan sebagai root
    if os.geteuid() != 0:
        print("[ERROR] Program harus dijalankan dengan sudo atau root.")
        print("Gunakan perintah: sudo python3 monitor_interface.py")
        return

    print("=== PROGRAM MONITOR INTERFACE ===")
    print("Program ini akan memeriksa status interface setiap 10 detik.")
    print("Jika ada interface DOWN (selain lo), akan dinonaktifkan otomatis.\n")

    try:
        while True:
            interfaces = get_interfaces_status()
            show_interfaces(interfaces)

            for name, status in interfaces.items():
                if name == "lo":
                    continue  # Abaikan loopback
                if status == "DOWN":
                    disable_interface(name)

            print("Menunggu 10 detik untuk pengecekan berikutnya...\n")
            time.sleep(10)

    except KeyboardInterrupt:
        print("\n[INFO] Program dihentikan oleh pengguna.")


if __name__ == "__main__":
    main()
