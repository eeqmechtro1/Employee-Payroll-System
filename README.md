# Employee-Payroll-System

#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <vector>
#include <iomanip>
#include <cstdlib>
#include <cstdio>
#include <windows.h>

using namespace std;

// ==================== GLOBAL VARIABLES ====================

string DB_FILENAME = "payroll_database.csv";

string HEADERS =
"Employee_ID,Employee_Name,Designation,Salary,Department,Email,Phone,Date_Joined";

// ==================== FUNCTION DECLARATIONS ====================

void initializeDatabase();
bool isUnique(string id);
void appendRecord(string data);
string searchByID(string id);
void updateRecord(string id, string newData);
void deleteRecord(string id);
void displayAllRecords();
int countRecords();
vector<string> parseCSVLine(string line);
void displayEmployeeDetails(string record);
void mainMenu();
void generateSampleData();

bool authenticateESP32();

// ==================== DISPLAY UTILITIES ====================

void clearScreen() {
    system("cls");
}

void printHeader(const string& text) {
    cout << "\n";
    cout << "+--------------------------------------------------------------------------------+\n";
    cout << "¦" << setw(83) << left << ("  " + text) << "¦\n";
    cout << "+--------------------------------------------------------------------------------+\n";
}

void printSubHeader(const string& text) {
    cout << "\n  +------------------------------------------------------------------------------+\n";
    cout << "  ¦ " << setw(79) << left << text << "¦\n";
    cout << "  +------------------------------------------------------------------------------+\n\n";
}

void printDivider() {
    cout << "  ---------------------------------------------------------------------------------\n";
}

void printSuccess(const string& msg) {
    cout << "\n  ? " << msg << "\n";
}

void printError(const string& msg) {
    cout << "\n  ? " << msg << "\n";
}

void printInfo(const string& msg) {
    cout << "  ? " << msg << "\n";
}

void pauseScreen() {
    cout << "\n  Press any key to continue...";
    system("pause > nul");
}

// ==================== ESP32 AUTHENTICATION ====================

bool authenticateESP32() {

    HANDLE hSerial = CreateFile(
        "\\\\.\\COM5",
        GENERIC_READ | GENERIC_WRITE,
        0,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );

    if (hSerial == INVALID_HANDLE_VALUE) {

        printError("ESP32 NOT CONNECTED!");

        return false;
    }

    DCB dcbSerialParams = {0};

    dcbSerialParams.DCBlength = sizeof(dcbSerialParams);

    if (!GetCommState(hSerial, &dcbSerialParams)) {

        printError("Failed to get COM state");

        CloseHandle(hSerial);

        return false;
    }

    dcbSerialParams.BaudRate = CBR_115200;
    dcbSerialParams.ByteSize = 8;
    dcbSerialParams.StopBits = ONESTOPBIT;
    dcbSerialParams.Parity = NOPARITY;

    if (!SetCommState(hSerial, &dcbSerialParams)) {

        printError("Failed to set COM state");

        CloseHandle(hSerial);

        return false;
    }

    COMMTIMEOUTS timeouts = {0};

    timeouts.ReadIntervalTimeout = 50;
    timeouts.ReadTotalTimeoutConstant = 50;
    timeouts.ReadTotalTimeoutMultiplier = 10;

    SetCommTimeouts(hSerial, &timeouts);

    DWORD bytesWritten;

    char request = 'R';

    WriteFile(hSerial, &request, 1, &bytesWritten, NULL);

    Sleep(1000);

    char buffer[50] = {0};

    DWORD bytesRead;

    if (ReadFile(hSerial, buffer, sizeof(buffer)-1, &bytesRead, NULL)) {

        buffer[bytesRead] = '\0';

        string receivedKey(buffer);

        if (receivedKey.find("HEXA_3") != string::npos) {

            printSuccess("ACCESS GRANTED");

            CloseHandle(hSerial);

            return true;
        }
    }

    CloseHandle(hSerial);

    printError("ACCESS DENIED");

    return false;
}

// ==================== MAIN FUNCTION ====================


int main() {

    clearScreen();

    printHeader("EMPLOYEE PAYROLL MANAGEMENT SYSTEM");

    printInfo("Authenticating with ESP32...");

    if (!authenticateESP32()) {

        pauseScreen();

        return 0;
    }

    initializeDatabase();

    mainMenu();

    return 0;
}

// ==================== INITIALIZE DATABASE ====================

void initializeDatabase() {

    ifstream file(DB_FILENAME.c_str());

    if (!file.good()) {

        ofstream out(DB_FILENAME.c_str());

        out << HEADERS << "\n";

        out.close();

        generateSampleData();
    }

    file.close();
}

// ==================== GENERATE SAMPLE DATA ====================

void generateSampleData() {

    vector<string> data;

    // IT Department - 20 employees
    data.push_back("EMP_001,Alice Johnson,Software Engineer,85000,IT,alice.johnson@company.com,5551001,2020-01-15");
    data.push_back("EMP_002,Bob Smith,Senior Developer,95000,IT,bob.smith@company.com,5551002,2019-03-22");
    data.push_back("EMP_003,Carol White,DevOps Engineer,90000,IT,carol.white@company.com,5551003,2020-06-10");
    data.push_back("EMP_004,David Brown,Database Admin,88000,IT,david.brown@company.com,5551004,2021-01-05");
    data.push_back("EMP_005,Emily Davis,IT Support,55000,IT,emily.davis@company.com,5551005,2021-07-12");
    data.push_back("EMP_006,Frank Miller,Network Admin,92000,IT,frank.miller@company.com,5551006,2020-02-18");
    data.push_back("EMP_007,Grace Lee,Cloud Architect,105000,IT,grace.lee@company.com,5551007,2019-11-30");
    data.push_back("EMP_008,Henry Wilson,System Admin,82000,IT,henry.wilson@company.com,5551008,2021-04-20");
    data.push_back("EMP_009,Iris Martinez,QA Engineer,75000,IT,iris.martinez@company.com,5551009,2020-08-14");
    data.push_back("EMP_010,Jack Taylor,Tech Lead,98000,IT,jack.taylor@company.com,5551010,2018-09-25");
    data.push_back("EMP_011,Karen Anderson,Security Engineer,96000,IT,karen.anderson@company.com,5551011,2019-12-10");
    data.push_back("EMP_012,Leo Thomas,Junior Developer,65000,IT,leo.thomas@company.com,5551012,2022-01-08");
    data.push_back("EMP_013,Mona Jackson,Full Stack Dev,87000,IT,mona.jackson@company.com,5551013,2020-05-19");
    data.push_back("EMP_014,Nathan White,Mobile Developer,84000,IT,nathan.white@company.com,5551014,2021-03-11");
    data.push_back("EMP_015,Olivia Harris,Data Engineer,91000,IT,olivia.harris@company.com,5551015,2020-10-22");
    data.push_back("EMP_016,Paul Martin,Infrastructure Eng,93000,IT,paul.martin@company.com,5551016,2019-07-14");
    data.push_back("EMP_017,Quinn Thompson,Software Architect,110000,IT,quinn.thompson@company.com,5551017,2018-05-20");
    data.push_back("EMP_018,Rachel Garcia,UX Developer,79000,IT,rachel.garcia@company.com,5551018,2021-02-09");
    data.push_back("EMP_019,Steve Rodriguez,Backend Engineer,89000,IT,steve.rodriguez@company.com,5551019,2020-11-16");
    data.push_back("EMP_020,Tina Lopez,Frontend Developer,82000,IT,tina.lopez@company.com,5551020,2021-06-03");

    // HR Department - 15 employees
    data.push_back("EMP_021,Uma Patel,HR Manager,75000,HR,uma.patel@company.com,5551021,2019-08-20");
    data.push_back("EMP_022,Victor Chang,Recruiter,68000,HR,victor.chang@company.com,5551022,2020-04-10");
    data.push_back("EMP_023,Wendy Kumar,HR Specialist,62000,HR,wendy.kumar@company.com,5551023,2021-01-25");
    data.push_back("EMP_024,Xavier Santos,Payroll Officer,65000,HR,xavier.santos@company.com,5551024,2020-09-14");
    data.push_back("EMP_025,Yara Khan,Training Coordinator,59000,HR,yara.khan@company.com,5551025,2021-03-30");
    data.push_back("EMP_026,Zack Ross,Benefits Admin,61000,HR,zack.ross@company.com,5551026,2020-12-08");
    data.push_back("EMP_027,Amy Foster,Employee Relations,67000,HR,amy.foster@company.com,5551027,2019-10-15");
    data.push_back("EMP_028,Brett Cohen,Compensation Analyst,64000,HR,brett.cohen@company.com,5551028,2021-05-22");
    data.push_back("EMP_029,Chloe Pierce,Recruitment Lead,76000,HR,chloe.pierce@company.com,5551029,2019-06-18");
    data.push_back("EMP_030,Derek Wells,HR Coordinator,58000,HR,derek.wells@company.com,5551030,2021-08-10");
    data.push_back("EMP_031,Evelyn Brooks,Training Manager,72000,HR,evelyn.brooks@company.com,5551031,2020-02-26");
    data.push_back("EMP_032,Felix Day,Talent Acquisition,69000,HR,felix.day@company.com,5551032,2020-07-12");
    data.push_back("EMP_033,Gina Morris,Compliance Officer,70000,HR,gina.morris@company.com,5551033,2019-11-22");
    data.push_back("EMP_034,Hunter Gray,HR Analyst,63000,HR,hunter.gray@company.com,5551034,2021-04-05");
    data.push_back("EMP_035,Ivy Stone,Culture Manager,66000,HR,ivy.stone@company.com,5551035,2020-08-19");

    // Finance Department - 18 employees
    data.push_back("EMP_036,Jack Newman,Finance Manager,82000,Finance,jack.newman@company.com,5551036,2019-05-15");
    data.push_back("EMP_037,Julia Burns,Accountant,68000,Finance,julia.burns@company.com,5551037,2020-03-22");
    data.push_back("EMP_038,Kevin Bell,Senior Accountant,78000,Finance,kevin.bell@company.com,5551038,2019-09-11");
    data.push_back("EMP_039,Laura Hunt,Financial Analyst,75000,Finance,laura.hunt@company.com,5551039,2020-06-30");
    data.push_back("EMP_040,Marco Costa,CFO,150000,Finance,marco.costa@company.com,5551040,2015-01-10");
    data.push_back("EMP_041,Nina Russell,Auditor,71000,Finance,nina.russell@company.com,5551041,2020-11-08");
    data.push_back("EMP_042,Oscar Mason,Tax Specialist,77000,Finance,oscar.mason@company.com,5551042,2019-12-20");
    data.push_back("EMP_043,Piper Hayes,Bookkeeper,59000,Finance,piper.hayes@company.com,5551043,2021-02-14");
    data.push_back("EMP_044,Quinn Davis,Budget Analyst,73000,Finance,quinn.davis@company.com,5551044,2020-08-05");
    data.push_back("EMP_045,Ryan Price,Payroll Manager,72000,Finance,ryan.price@company.com,5551045,2020-04-28");
    data.push_back("EMP_046,Sophie Bennett,Financial Planner,79000,Finance,sophie.bennett@company.com,5551046,2019-07-09");
    data.push_back("EMP_047,Thomas Dunn,Controller,88000,Finance,thomas.dunn@company.com,5551047,2018-10-16");
    data.push_back("EMP_048,Uma Singh,Investment Analyst,81000,Finance,uma.singh@company.com,5551048,2019-04-23");
    data.push_back("EMP_049,Vincent Murphy,Junior Accountant,58000,Finance,vincent.murphy@company.com,5551049,2021-06-15");
    data.push_back("EMP_050,Wilma Shaw,Risk Analyst,76000,Finance,wilma.shaw@company.com,5551050,2020-09-27");
    data.push_back("EMP_051,Xavier Hunt,Accounts Payable,62000,Finance,xavier.hunt@company.com,5551051,2021-01-20");
    data.push_back("EMP_052,Yasmine Davis,Accounts Receivable,63000,Finance,yasmine.davis@company.com,5551052,2020-12-03");
    data.push_back("EMP_053,Zachary Fox,Financial Controller,85000,Finance,zachary.fox@company.com,5551053,2019-08-14");

    // Marketing Department - 17 employees
    data.push_back("EMP_054,Ava Green,Marketing Manager,78000,Marketing,ava.green@company.com,5551054,2020-02-10");
    data.push_back("EMP_055,Bradley King,Digital Marketer,70000,Marketing,bradley.king@company.com,5551055,2020-07-18");
    data.push_back("EMP_056,Camilla Wright,SEO Specialist,69000,Marketing,camilla.wright@company.com,5551056,2021-01-30");
    data.push_back("EMP_057,Damian Lopez,Content Creator,65000,Marketing,damian.lopez@company.com,5551057,2021-03-12");
    data.push_back("EMP_058,Elena Scott,Brand Manager,75000,Marketing,elena.scott@company.com,5551058,2019-11-05");
    data.push_back("EMP_059,Fabian Green,Social Media Manager,67000,Marketing,fabian.green@company.com,5551059,2020-05-22");
    data.push_back("EMP_060,Gloria Adams,Product Marketer,73000,Marketing,gloria.adams@company.com,5551060,2019-09-18");
    data.push_back("EMP_061,Harrison Clark,Email Marketer,64000,Marketing,harrison.clark@company.com,5551061,2021-04-08");
    data.push_back("EMP_062,Iris Morgan,Marketing Analyst,68000,Marketing,iris.morgan@company.com,5551062,2020-08-25");
    data.push_back("EMP_063,Jake Rivers,Campaign Manager,72000,Marketing,jake.rivers@company.com,5551063,2020-03-15");
    data.push_back("EMP_064,Keira Stone,Copywriter,62000,Marketing,keira.stone@company.com,5551064,2021-06-02");
    data.push_back("EMP_065,Liam Hayes,Market Researcher,70000,Marketing,liam.hayes@company.com,5551065,2020-10-12");
    data.push_back("EMP_066,Molly Jensen,Brand Specialist,66000,Marketing,molly.jensen@company.com,5551066,2021-02-20");
    data.push_back("EMP_067,Nolan Chase,Graphic Designer,65000,Marketing,nolan.chase@company.com,5551067,2020-11-08");
    data.push_back("EMP_068,Olive Rivers,Video Producer,71000,Marketing,olive.rivers@company.com,5551068,2019-12-14");
    data.push_back("EMP_069,Parker Lane,PR Manager,77000,Marketing,parker.lane@company.com,5551069,2019-08-30");
    data.push_back("EMP_070,Quinn Roberts,Event Coordinator,61000,Marketing,quinn.roberts@company.com,5551070,2021-05-18");

    // Sales Department - 15 employees
    data.push_back("EMP_071,Rachel Nelson,Sales Manager,80000,Sales,rachel.nelson@company.com,5551071,2019-06-20");
    data.push_back("EMP_072,Samuel Carter,Sales Executive,72000,Sales,samuel.carter@company.com,5551072,2020-04-15");
    data.push_back("EMP_073,Tanya Phillips,Account Executive,75000,Sales,tanya.phillips@company.com,5551073,2020-08-10");
    data.push_back("EMP_074,Ulrich Campbell,Sales Rep,65000,Sales,ulrich.campbell@company.com,5551074,2021-02-25");
    data.push_back("EMP_075,Vanessa Parker,Inside Sales,68000,Sales,vanessa.parker@company.com,5551075,2020-09-14");
    data.push_back("EMP_076,Wesley Evans,Business Developer,78000,Sales,wesley.evans@company.com,5551076,2019-10-22");
    data.push_back("EMP_077,Xenia Harrison,Key Account Manager,81000,Sales,xenia.harrison@company.com,5551077,2019-05-09");
    data.push_back("EMP_078,Yasmin Hughes,Territory Manager,76000,Sales,yasmin.hughes@company.com,5551078,2020-01-30");
    data.push_back("EMP_079,Zachary Knight,Sales Analyst,69000,Sales,zachary.knight@company.com,5551079,2020-11-05");
    data.push_back("EMP_080,Aurora Long,Enterprise Sales,85000,Sales,aurora.long@company.com,5551080,2019-03-18");
    data.push_back("EMP_081,Brandon Price,Channel Manager,77000,Sales,brandon.price@company.com,5551081,2020-06-12");
    data.push_back("EMP_082,Cassandra Reid,Quota Manager,74000,Sales,cassandra.reid@company.com,5551082,2020-12-08");
    data.push_back("EMP_083,Derek Sanchez,Telemarketer,58000,Sales,derek.sanchez@company.com,5551083,2021-07-22");
    data.push_back("EMP_084,Elena Taylor,Sales Coordinator,62000,Sales,elena.taylor@company.com,5551084,2021-01-15");
    data.push_back("EMP_085,Fredrick Young,Regional Director,95000,Sales,fredrick.young@company.com,5551085,2018-09-25");

    // Operations Department - 15 employees
    data.push_back("EMP_086,Giselle Adams,Operations Manager,79000,Operations,giselle.adams@company.com,5551086,2019-07-11");
    data.push_back("EMP_087,Harrison Baker,Process Analyst,68000,Operations,harrison.baker@company.com,5551087,2020-05-20");
    data.push_back("EMP_088,Isabelle Carter,Supply Chain Manager,77000,Operations,isabelle.carter@company.com,5551088,2019-11-14");
    data.push_back("EMP_089,Jerome Davis,Logistics Officer,71000,Operations,jerome.davis@company.com,5551089,2020-03-09");
    data.push_back("EMP_090,Kiersten Edwards,Warehouse Manager,73000,Operations,kiersten.edwards@company.com,5551090,2020-08-18");
    data.push_back("EMP_091,Lester Foster,Procurement Officer,70000,Operations,lester.foster@company.com,5551091,2021-02-04");
    data.push_back("EMP_092,Magnolia Gray,Quality Manager,76000,Operations,magnolia.gray@company.com,5551092,2019-09-27");
    data.push_back("EMP_093,Nicholas Harris,Compliance Manager,74000,Operations,nicholas.harris@company.com,5551093,2020-06-15");
    data.push_back("EMP_094,Olivia Johnson,Facilities Manager,69000,Operations,olivia.johnson@company.com,5551094,2020-10-22");
    data.push_back("EMP_095,Penelope King,Operations Analyst,67000,Operations,penelope.king@company.com,5551095,2021-04-10");
    data.push_back("EMP_096,Quinton Lee,Maintenance Manager,72000,Operations,quinton.lee@company.com,5551096,2020-01-28");
    data.push_back("EMP_097,Rosalinda Martinez,Inventory Clerk,58000,Operations,rosalinda.martinez@company.com,5551097,2021-05-19");
    data.push_back("EMP_098,Silas Nelson,Safety Officer,70000,Operations,silas.nelson@company.com,5551098,2019-12-10");
    data.push_back("EMP_099,Tanya Oliver,Production Manager,81000,Operations,tanya.oliver@company.com,5551099,2019-04-22");
    data.push_back("EMP_100,Ulysses Park,Operations Director,92000,Operations,ulysses.park@company.com,5551100,2018-11-30");

    for (size_t i = 0; i < data.size(); i++) {
        ofstream file(DB_FILENAME.c_str(), ios::app);
        file << data[i] << "\n";
        file.close();
    }
}

// ==================== PARSE CSV ====================

vector<string> parseCSVLine(string line) {

    vector<string> fields;

    stringstream ss(line);

    string temp;

    while (getline(ss, temp, ',')) {

        fields.push_back(temp);
    }

    return fields;
}

// ==================== CHECK UNIQUE ====================

bool isUnique(string id) {

    ifstream file(DB_FILENAME.c_str());

    string line;

    while (getline(file, line)) {

        vector<string> f = parseCSVLine(line);

        if (!f.empty() && f[0] == id) {

            return false;
        }
    }

    return true;
}

// ==================== APPEND ====================

void appendRecord(string data) {

    vector<string> f = parseCSVLine(data);

    if (f.empty() || f[0] == "") return;

    if (!isUnique(f[0])) {

        printError("Employee ID already exists!");

        return;
    }

    ofstream file(DB_FILENAME.c_str(), ios::app);

    file << data << "\n";

    printSuccess("Record Added Successfully!");
}

// ==================== SEARCH ====================

string searchByID(string id) {

    ifstream file(DB_FILENAME.c_str());

    string line;

    while (getline(file, line)) {

        vector<string> f = parseCSVLine(line);

        if (!f.empty() && f[0] == id) {

            return line;
        }
    }

    return "";
}

// ==================== UPDATE ====================

void updateRecord(string id, string newData) {

    ifstream in(DB_FILENAME.c_str());

    ofstream temp("temp.csv");

    string line;

    bool found = false;

    while (getline(in, line)) {

        vector<string> f = parseCSVLine(line);

        if (!f.empty() && f[0] == id) {

            temp << newData << "\n";

            found = true;
        }
        else {

            temp << line << "\n";
        }
    }

    in.close();

    temp.close();

    remove(DB_FILENAME.c_str());

    rename("temp.csv", DB_FILENAME.c_str());

    if (found)
        printSuccess("Updated Successfully");
    else
        printError("ID Not Found");
}

// ==================== DELETE ====================

void deleteRecord(string id) {

    ifstream in(DB_FILENAME.c_str());

    ofstream temp("temp.csv");

    string line;

    bool found = false;

    while (getline(in, line)) {

        vector<string> f = parseCSVLine(line);

        if (!f.empty() && f[0] == id) {

            found = true;
        }
        else {

            temp << line << "\n";
        }
    }

    in.close();

    temp.close();

    remove(DB_FILENAME.c_str());

    rename("temp.csv", DB_FILENAME.c_str());

    if (found)
        printSuccess("Deleted Successfully");
    else
        printError("ID Not Found");
}

// ==================== DISPLAY ====================

void displayAllRecords() {

    clearScreen();

    ifstream file(DB_FILENAME.c_str());

    string line;

    printSubHeader("ALL EMPLOYEE RECORDS");

    cout << "  " << setw(10) << left << "ID"
         << setw(20) << left << "Name"
         << setw(25) << left << "Designation"
         << setw(10) << right << "Salary"
         << setw(15) << right << "Department\n";

    printDivider();

    bool first = true;
    int count = 0;
    while (getline(file, line)) {

        if (first) {
            first = false;
            continue;
        }

        vector<string> fields = parseCSVLine(line);

        if (fields.size() >= 5) {
            cout << "  " << setw(10) << left << fields[0]
                 << setw(20) << left << fields[1]
                 << setw(25) << left << fields[2]
                 << setw(10) << right << fields[3]
                 << setw(15) << right << fields[4] << "\n";
            count++;

            if (count % 15 == 0) {
                cout << "\n";
            }
        }
    }

    printDivider();

    cout << "\n  Total Records: " << count << "\n";

    file.close();

    pauseScreen();
}

// ==================== COUNT ====================

int countRecords() {

    ifstream file(DB_FILENAME.c_str());

    string line;

    int count = -1;

    while (getline(file, line)) {

        count++;
    }

    return count;
}

// ==================== DISPLAY DETAILS ====================

void displayEmployeeDetails(string record) {

    clearScreen();

    vector<string> f = parseCSVLine(record);

    if (f.size() < 8) return;

    printSubHeader("EMPLOYEE DETAILS");

    cout << "  Employee ID      : " << f[0] << "\n";
    cout << "  Name             : " << f[1] << "\n";
    cout << "  Designation      : " << f[2] << "\n";
    cout << "  Salary           : $" << f[3] << "\n";
    cout << "  Department       : " << f[4] << "\n";
    cout << "  Email            : " << f[5] << "\n";
    cout << "  Phone            : " << f[6] << "\n";
    cout << "  Date Joined      : " << f[7] << "\n";

    pauseScreen();
}

// ==================== MENU ====================

void mainMenu() {

    int ch;

    do {

        clearScreen();

        printHeader("MAIN MENU");

        cout << "  1. Display All Records\n";
        cout << "  2. Search Employee\n";
        cout << "  3. Add Employee\n";
        cout << "  4. Update Employee\n";
        cout << "  5. Delete Employee\n";
        cout << "  0. Exit\n";

        printDivider();

        cout << "  Enter Choice (0-5): ";

        cin >> ch;

        cin.ignore();

        if (ch == 1) {

            displayAllRecords();
        }

        else if (ch == 2) {

            clearScreen();
            printSubHeader("SEARCH EMPLOYEE");

            string id;

            cout << "  Enter Employee ID: ";

            getline(cin, id);

            string r = searchByID(id);

            if (r != "")
                displayEmployeeDetails(r);
            else {
                printError("Employee Not Found");
                pauseScreen();
            }
        }

        else if (ch == 3) {

            clearScreen();
            printSubHeader("ADD NEW EMPLOYEE");

            string id, name, des, sal, dep, email, phone, date;

            cout << "  ID: ";
            getline(cin, id);

            cout << "  Name: ";
            getline(cin, name);

            cout << "  Designation: ";
            getline(cin, des);

            cout << "  Salary: ";
            getline(cin, sal);

            cout << "  Department: ";
            getline(cin, dep);

            cout << "  Email: ";
            getline(cin, email);

            cout << "  Phone: ";
            getline(cin, phone);

            cout << "  Date Joined (YYYY-MM-DD): ";
            getline(cin, date);

            string rec =
            id + "," +
            name + "," +
            des + "," +
            sal + "," +
            dep + "," +
            email + "," +
            phone + "," +
            date;

            appendRecord(rec);

            pauseScreen();
        }

        else if (ch == 4) {

            clearScreen();
            printSubHeader("UPDATE EMPLOYEE");

            string id;

            cout << "  Enter Employee ID to Update: ";

            getline(cin, id);

            string found = searchByID(id);

            if (found == "") {
                printError("Employee Not Found");
                pauseScreen();
                continue;
            }

            clearScreen();
            printSubHeader("UPDATE EMPLOYEE - ENTER NEW DATA");

            string rec;

            cout << "  Enter New Full Record (CSV Format, 8 fields):\n";
            cout << "  Format: ID,Name,Designation,Salary,Department,Email,Phone,DateJoined\n\n";
            cout << "  New Record: ";

            getline(cin, rec);

            updateRecord(id, rec);

            pauseScreen();
        }

        else if (ch == 5) {

            clearScreen();
            printSubHeader("DELETE EMPLOYEE");

            string id;

            cout << "  Enter Employee ID to Delete: ";

            getline(cin, id);

            string found = searchByID(id);

            if (found == "") {
                printError("Employee Not Found");
                pauseScreen();
                continue;
            }

            cout << "\n  Confirm deletion? (yes/no): ";
            string confirm;
            getline(cin, confirm);

            if (confirm == "yes" || confirm == "YES") {
                deleteRecord(id);
            } else {
                printInfo("Deletion cancelled");
            }

            pauseScreen();
        }

        else if (ch != 0) {
            clearScreen();
            printError("Invalid Choice! Please enter 0-5");
            pauseScreen();
        }

    } while (ch != 0);

    clearScreen();
    printHeader("THANK YOU FOR USING THE SYSTEM");
    cout << "\n  Exiting...\n\n";
}
