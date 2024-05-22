#include <iostream>
#include <windows.h>
#include <tlhelp32.h>
#include <string>
#include <thread>
#include <chrono>

const SIZE_T maxMemoryUsage = 750 * 1024 * 1024;
const SIZE_T minMemoryUsage = 1 * 1024 * 1024;

void limitMemoryUsage(DWORD processId) {
    HANDLE hProcess = OpenProcess(PROCESS_SET_QUOTA, FALSE, processId);
    if (hProcess != NULL) {
        SetProcessWorkingSetSize(hProcess, minMemoryUsage, maxMemoryUsage);
        CloseHandle(hProcess);
    }
    else {
        std::wcout << "Fail to limit ram for one tab of wow!" << std::endl;
    }
}

void monitorWorldOfWarcraft() {
    std::wcout << "Checking new tabs" << std::endl;
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hSnapshot != INVALID_HANDLE_VALUE) {
        PROCESSENTRY32 pe32;
        pe32.dwSize = sizeof(PROCESSENTRY32);

        if (Process32First(hSnapshot, &pe32)) {
            do {
                std::wstring processName = pe32.szExeFile;
                if (processName.find(L"Wow.exe") != std::string::npos || processName.find(L"WowVoiceProxy.exe") != std::string::npos) {
                    std::wcout << "Found World of Warcraft process with ID: " << pe32.th32ProcessID << std::endl;
                    limitMemoryUsage(pe32.th32ProcessID);
                }
            } while (Process32Next(hSnapshot, &pe32));
        }
        else {
            std::wcout << "Cant get list of new apps" << std::endl;
        }
        CloseHandle(hSnapshot);
    }
    else {
        std::wcout << "Cant get list of new apps #2" << std::endl;
    }
}

int main() {
    std::wcout << "Program started!" << std::endl;
    std::thread monitorThread(monitorWorldOfWarcraft);

    monitorThread.join();

    return 0;
