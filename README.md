# Cpp
#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <iomanip>

using namespace std;

// === WEEK 1: STUDENT CLASS ===
class Student {
public:
    string name;
    string indexNumber;

    Student(string n = "", string id = "") : name(n), indexNumber(id) {}

    void display() const {
        cout << left << setw(20) << name << setw(15) << indexNumber << endl;
    }
};

// === WEEK 2 & 3: ATTENDANCE SESSION & LOGIC ===
struct AttendanceRecord {
    string studentIndex;
    string status; // Present, Absent, Late
};

class AttendanceSession {
public:
    string courseCode;
    string date;
    string startTime;
    vector<AttendanceRecord> records;

    AttendanceSession(string code, string d, string time) 
        : courseCode(code), date(d), startTime(time) {}

    void markAttendance(const vector<Student>& students) {
        records.clear();
        for (const auto& student : students) {
            int choice;
            cout << "Mark " << student.name << " (1: Present, 2: Absent, 3: Late): ";
            cin >> choice;
            
            string status = (choice == 1) ? "Present" : (choice == 3) ? "Late" : "Absent";
            records.push_back({student.indexNumber, status});
        }
    }

    void displaySummary() const {
        int p = 0, a = 0, l = 0;
        cout << "\n--- Attendance for " << courseCode << " (" << date << ") ---\n";
        for (const auto& r : records) {
            cout << "Index: " << r.studentIndex << " | Status: " << r.status << endl;
            if (r.status == "Present") p++;
            else if (r.status == "Late") l++;
            else a++;
        }
        cout << "\nSummary: Present: " << p << " | Absent: " << a << " | Late: " << l << endl;
    }
};

// === WEEK 4: FILE STORAGE & REFACTORING ===
void saveStudents(const vector<Student>& students) {
    ofstream outFile("students.txt");
    for (const auto& s : students) {
        outFile << s.name << "," << s.indexNumber << endl;
    }
    outFile.close();
}

void loadStudents(vector<Student>& students) {
    ifstream inFile("students.txt");
    string name, id;
    while (getline(inFile, name, ',') && getline(inFile, id)) {
        students.push_back(Student(name, id));
    }
    inFile.close();
}

// === MAIN MENU SYSTEM ===
int main() {
    vector<Student> students;
    loadStudents(students); // Load existing data on startup
    int choice;

    do {
        cout << "\n--- DIGITAL ATTENDANCE SYSTEM ---\n";
        cout << "1. Register Student\n2. View Students\n3. Create Attendance Session\n4. Exit\nChoice: ";
        cin >> choice;

        if (choice == 1) {
            string name, id;
            cout << "Enter Name: "; cin.ignore(); getline(cin, name);
            cout << "Enter Index Number: "; cin >> id;
            students.push_back(Student(name, id));
            saveStudents(students);
        } 
        else if (choice == 2) {
            cout << left << setw(20) << "\nName" << setw(15) << "Index Number" << endl;
            for (const auto& s : students) s.display();
        } 
        else if (choice == 3) {
            if (students.empty()) {
                cout << "Register students first!\n";
                continue;
            }
            string code, date, time;
            cout << "Course Code: "; cin >> code;
            cout << "Date (YYYY_MM_DD): "; cin >> date;
            cout << "Start Time: "; cin >> time;

            AttendanceSession session(code, date, time);
            session.markAttendance(students);
            session.displaySummary();

            // Save session to unique file
            string filename = "session_" + code + "_" + date + ".txt";
            ofstream sFile(filename);
            sFile << "Course: " << code << " Date: " << date << "\n";
            for (const auto& r : session.records) {
                sFile << r.studentIndex << " : " << r.status << "\n";
            }
            sFile.close();
            cout << "Saved to " << filename << endl;
        }
    } while (choice != 4);

    return 0;
}
