# Startup-codes
The following codes are documented in order to track my progress throughout software engineering
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>

using namespace std;

class Person {
private:
    string name;
    int birthYear;
    string motherName;
    string fatherName;
    vector<string> childrenNames;

public:
    Person(string n, int by) : name(n), birthYear(by) {
        motherName = "";
        fatherName = "";
    }

    string getName() const {
        return name;
    }

    int getBirthYear() const {
        return birthYear;
    }

    string getMotherName() const {
        return motherName;
    }

    string getFatherName() const {
        return fatherName;
    }

    vector<string> getChildrenNames() const {
        return childrenNames;
    }

    void setMotherName(const string& mName) {
        motherName = mName;
    }

    void setFatherName(const string& fName) {
        fatherName = fName;
    }

    void addChildName(const string& childName) {
        childrenNames.push_back(childName);
    }
};

class FamilyTree {
private:
    vector<Person> people;

    int findPersonIndex(const string& name) const {
        for (size_t i = 0; i < people.size(); ++i) {
            string currentName = people[i].getName();
            if (currentName == name) {
                return static_cast<int>(i);
            }
        }
        return -1;
    }

public:
    FamilyTree() {}

    bool loadFromFile(const string& peopleFile, const string& relationshipsFile) {
        ifstream peopleStream;
        peopleStream.open(peopleFile);

        if (!peopleStream.is_open()) {
            return false;
        }

        string line;
        while (getline(peopleStream, line)) {
            istringstream iss(line);
            string name;
            int birthYear;

            if (iss >> name >> birthYear) {
                Person newPerson(name, birthYear);
                people.push_back(newPerson);
            }
        }
        peopleStream.close();

        ifstream relationshipsStream(relationshipsFile);
        if (!relationshipsStream.is_open()) {
            return false;
        }

        while (getline(relationshipsStream, line)) {
            istringstream iss(line);
            string father, mother, child;

            if (iss >> father >> mother >> child) {
                if (father != "-" && child != "-") {
                    setParentChild(father, child, false);
                }
                if (mother != "-" && child != "-") {
                    setParentChild(mother, child, true);
                }
            }
        }
        relationshipsStream.close();

        return true;
    }

    void addPerson(string name, int birthYear) {
        if (findPersonIndex(name) != -1) {
            cout << "Person " << name << " already exists." << endl;
            return;
        }

        Person newPerson(name, birthYear);
        people.push_back(newPerson);
    }

    bool setParentChild(string parentName, string childName, bool isMother) {
        int parentIndex = findPersonIndex(parentName);
        int childIndex = findPersonIndex(childName);

        if (parentIndex == -1) {
            cout << "Person not found: " << parentName << endl;
            return false;
        }

        if (childIndex == -1) {
            cout << "Person not found: " << childName << endl;
            return false;
        }

        people[parentIndex].addChildName(childName);

        if (isMother) {
            people[childIndex].setMotherName(parentName);
        }
        else {
            people[childIndex].setFatherName(parentName);
        }

        return true;
    }

    vector<string> findAncestors(const string& name, int generations) const {
        vector<string> result;
        int personIndex = findPersonIndex(name);
        if (personIndex == -1 || generations <= 0) {
            return result;
        }

        const Person& person = people[personIndex];
        string motherName = person.getMotherName();
        string fatherName = person.getFatherName();

        if (!motherName.empty()) {
            result.push_back("Generation 1: " + motherName);
        }
        if (!fatherName.empty()) {
            result.push_back("Generation 1: " + fatherName);
        }

        if (generations > 1) {
            if (!motherName.empty()) {
                vector<string> motherAncestors = findAncestors(motherName, generations - 1);
                for (const string& anc : motherAncestors) {
                    int gen = stoi(anc.substr(11, anc.find(':') - 11));
                    result.push_back("Generation " + to_string(gen + 1) + anc.substr(anc.find(':')));
                }
            }

            if (!fatherName.empty()) {
                vector<string> fatherAncestors = findAncestors(fatherName, generations - 1);
                for (const string& anc : fatherAncestors) {
                    int gen = stoi(anc.substr(11, anc.find(':') - 11));
                    result.push_back("Generation " + to_string(gen + 1) + anc.substr(anc.find(':')));
                }
            }
        }

        return result;
    }

    vector<string> findDescendants(const string& name, int generations) const {
        vector<string> result;
        int personIndex = findPersonIndex(name);
        if (personIndex == -1 || generations <= 0) {
            return result;
        }

        const Person& person = people[personIndex];
        vector<string> childrenNames = person.getChildrenNames();

        if (!childrenNames.empty()) {
            string genStr = "Generation 1: ";
            for (size_t i = 0; i < childrenNames.size(); i++) {
                genStr += childrenNames[i];
                if (i < childrenNames.size() - 1) {
                    genStr += ", ";
                }
            }
            result.push_back(genStr);

            if (generations > 1) {
                for (const string& childName : childrenNames) {
                    vector<string> childDescendants = findDescendants(childName, generations - 1);
                    for (const string& desc : childDescendants) {
                        int gen = stoi(desc.substr(11, desc.find(':') - 11));
                        result.push_back("Generation " + to_string(gen + 1) + desc.substr(desc.find(':')));
                    }
                }
            }
        }

        return result;
    }

    void displayPerson(const string& name) const {
        int personIndex = findPersonIndex(name);
        if (personIndex == -1) {
            cout << "Person not found: " << name << endl << endl;
            return;
        }

        const Person& person = people[personIndex];
        cout << "Name: " << person.getName() << endl;
        cout << "Birth Year: " << person.getBirthYear() << endl;

        string motherName = person.getMotherName();
        string fatherName = person.getFatherName();

        if (motherName.empty() && fatherName.empty()) {
            cout << "Parents: None" << endl;
        }
        else {
            cout << "Parents: ";
            if (!fatherName.empty()) {
                cout << fatherName << " (father)";
                if (!motherName.empty()) {
                    cout << ", ";
                }
            }
            if (!motherName.empty()) {
                cout << motherName << " (mother)";
            }
            cout << endl;
        }

        vector<string> children = person.getChildrenNames();
        if (children.empty()) {
            cout << "Children: None" << endl;
        }
        else {
            cout << "Children: ";
            for (size_t i = 0; i < children.size(); i++) {
                cout << children[i];
                if (i < children.size() - 1) {
                    cout << ", ";
                }
            }
            cout << endl;
        }
        cout << endl;
    }
};

void displayMenu() {
    cout << "Menu:" << endl;
    cout << "1. Display a person's information" << endl;
    cout << "2. Find ancestors of a person" << endl;
    cout << "3. Find descendants of a person" << endl;
    cout << "4. Add a new person" << endl;
    cout << "5. Add a parent-child relationship" << endl;
    cout << "6. Exit" << endl;
}

int main() {
    FamilyTree familyTree;

    string peopleFile, relationshipsFile;
    cout << "Enter people file name: ";
    cin >> peopleFile;
    cout << "Enter relationships file name: ";
    cin >> relationshipsFile;

    if (familyTree.loadFromFile(peopleFile, relationshipsFile)) {
        cout << "Family tree loaded successfully!" << endl;
    }
    else {
        cout << "Error loading files!" << endl;
        return 1;
    }
    cout << endl;

    int choice;
    string name, parentName, childName;
    int birthYear, generations;
    char parentType;

    while (true) {
        displayMenu();
        cout << "Enter choice: ";
        cin >> choice;

        if (choice == 1) {
            cout << "Enter person name: ";
            cin >> name;
            familyTree.displayPerson(name);
        }
        else if (choice == 2) {
            cout << "Enter person name: ";
            cin >> name;
            cout << "Enter number of generations to search: ";
            cin >> generations;

            vector<string> ancestors = familyTree.findAncestors(name, generations);
            cout << "Ancestors of " << name << " (up to " << generations << " generations):" << endl;

            if (ancestors.empty()) {
                cout << "No known ancestors for " << name << "." << endl;
            }
            else {
                for (const string& ancestor : ancestors) {
                    cout << ancestor << endl;
                }
            }
            cout << endl;
        }
        else if (choice == 3) {
            cout << "Enter person name: ";
            cin >> name;
            cout << "Enter number of generations to search: ";
            cin >> generations;

            vector<string> descendants = familyTree.findDescendants(name, generations);
            cout << "Descendants of " << name << " (up to " << generations << " generations):" << endl;

            if (descendants.empty()) {
                cout << "No known descendants for " << name << "." << endl;
            }
            else {
                for (const string& descendant : descendants) {
                    cout << descendant << endl;
                }
            }
            cout << endl;
        }
        else if (choice == 4) {
            cout << "Enter name: ";
            cin >> name;
            cout << "Enter birth year: ";
            cin >> birthYear;
            familyTree.addPerson(name, birthYear);
            cout << endl;
        }
        else if (choice == 5) {
            cout << "Enter parent name: ";
            cin >> parentName;
            cout << "Is this parent a mother (m) or father (f)? ";
            cin >> parentType;
            cout << "Enter child name (or type '-' for no child): ";
            cin >> childName;

            if (childName != "-") {
                bool isMother = (parentType == 'm' || parentType == 'M');
                if (familyTree.setParentChild(parentName, childName, isMother)) {
                    cout << "Relationship added successfully." << endl;
                }
                else {
                    cout << "Failed to add relationship." << endl;
                }
            }
            cout << endl;
        }
        else if (choice == 6) {
            cout << "Goodbye!" << endl;
            return 0;
        }
        else {
            cout << "Invalid choice. Please try again." << endl << endl;
        }
    }

    return 0;
}