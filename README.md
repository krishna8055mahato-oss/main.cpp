
#include <iostream>
#include <vector>
#include <string>
#include <iomanip>
#include <algorithm>
#include <ctime>
#include <map>

using namespace std;

// Enums for project management
enum Priority { LOW = 1, MEDIUM = 2, HIGH = 3, CRITICAL = 4 };
enum TaskStatus { PENDING = 0, IN_PROGRESS = 1, IN_REVIEW = 2, COMPLETED = 3, BLOCKED = 4 };
enum ProjectStatus { NOT_STARTED = 0, PLANNING = 1, ACTIVE = 2, ON_HOLD = 3, COMPLETED_P = 4 };

// Forward declarations
class Project;
class Task;
class TeamMember;

// Task class - represents individual work items
class Task {
private:
    int taskId;
    string title;
    string description;
    Priority priority;
    TaskStatus status;
    string assignedTo;
    string dueDate;
    int estimatedHours;
    int actualHours;
    
public:
    Task(int id, string t, string des, Priority p, string due, int hours)
        : taskId(id), title(t), description(des), priority(p), status(PENDING),
          dueDate(due), estimatedHours(hours), actualHours(0) {
        assignedTo = "Unassigned";
    }
    
    // Getters
    int getTaskId() const { return taskId; }
    string getTitle() const { return title; }
    TaskStatus getStatus() const { return status; }
    string getAssignedTo() const { return assignedTo; }
    Priority getPriority() const { return priority; }
    int getEstimatedHours() const { return estimatedHours; }
    int getActualHours() const { return actualHours; }
    string getDueDate() const { return dueDate; }
    
    // Setters
    void setStatus(TaskStatus s) { status = s; }
    void assignTo(string member) { assignedTo = member; }
    void setActualHours(int hours) { actualHours = hours; }
    void setPriority(Priority p) { priority = p; }
    
    // Display task details
    void display() const {
        cout << "\n  Task ID: " << taskId << " | " << title;
        cout << "\n  Description: " << description;
        cout << "\n  Status: ";
        
        switch(status) {
            case PENDING: cout << "PENDING"; break;
            case IN_PROGRESS: cout << "IN PROGRESS"; break;
            case IN_REVIEW: cout << "IN REVIEW"; break;
            case COMPLETED: cout << "COMPLETED"; break;
            case BLOCKED: cout << "BLOCKED"; break;
        }
        
        cout << " | Priority: ";
        switch(priority) {
            case LOW: cout << "LOW"; break;
            case MEDIUM: cout << "MEDIUM"; break;
            case HIGH: cout << "HIGH"; break;
            case CRITICAL: cout << "CRITICAL"; break;
        }
        
        cout << "\n  Assigned To: " << assignedTo;
        cout << "\n  Due Date: " << dueDate;
        cout << "\n  Estimated Hours: " << estimatedHours << " | Actual Hours: " << actualHours;
    }
};

// TeamMember class
class TeamMember {
private:
    string memberId;
    string name;
    string role;
    vector<int> assignedTasks;
    int tasksCompleted;
    
public:
    TeamMember(string id, string n, string r)
        : memberId(id), name(n), role(r), tasksCompleted(0) {}
    
    // Getters
    string getMemberId() const { return memberId; }
    string getName() const { return name; }
    string getRole() const { return role; }
    int getTasksCompleted() const { return tasksCompleted; }
    
    // Task management
    void assignTask(int taskId) {
        assignedTasks.push_back(taskId);
    }
    
    void completeTask(int taskId) {
        auto it = find(assignedTasks.begin(), assignedTasks.end(), taskId);
        if (it != assignedTasks.end()) {
            assignedTasks.erase(it);
            tasksCompleted++;
        }
    }
    
    int getAssignedTaskCount() const { return assignedTasks.size(); }
    
    void display() const {
        cout << "\n  ID: " << memberId << " | Name: " << name;
        cout << "\n  Role: " << role;
        cout << "\n  Assigned Tasks: " << assignedTasks.size();
        cout << "\n  Completed Tasks: " << tasksCompleted;
    }
};

// Project class
class Project {
private:
    int projectId;
    string projectName;
    string description;
    ProjectStatus status;
    string startDate;
    string endDate;
    vector<Task> tasks;
    vector<TeamMember> team;
    int taskIdCounter;
    
public:
    Project(int id, string name, string des, string start, string end)
        : projectId(id), projectName(name), description(des), status(NOT_STARTED),
          startDate(start), endDate(end), taskIdCounter(1000) {}
    
    // Getters
    int getProjectId() const { return projectId; }
    string getProjectName() const { return projectName; }
    ProjectStatus getStatus() const { return status; }
    int getTaskCount() const { return tasks.size(); }
    int getTeamSize() const { return team.size(); }
    
    // Status management
    void setStatus(ProjectStatus s) { status = s; }
    
    // Team management
    void addTeamMember(string id, string name, string role) {
        team.push_back(TeamMember(id, name, role));
        cout << "\n✓ Team member " << name << " added successfully!";
    }
    
    void removeTeamMember(string id) {
        auto it = find_if(team.begin(), team.end(),
                         [id](const TeamMember& m) { return m.getMemberId() == id; });
        if (it != team.end()) {
            team.erase(it);
            cout << "\n✓ Team member removed successfully!";
        }
    }
    
    // Task management
    void createTask(string title, string description, Priority priority, 
                   string dueDate, int estimatedHours) {
        taskIdCounter++;
        tasks.push_back(Task(taskIdCounter, title, description, priority, dueDate, estimatedHours));
        cout << "\n✓ Task created with ID: " << taskIdCounter;
    }
    
    void assignTask(int taskId, string memberId) {
        auto taskIt = find_if(tasks.begin(), tasks.end(),
                             [taskId](const Task& t) { return t.getTaskId() == taskId; });
        auto memberIt = find_if(team.begin(), team.end(),
                               [memberId](const TeamMember& m) { return m.getMemberId() == memberId; });
        
        if (taskIt != tasks.end() && memberIt != team.end()) {
            taskIt->assignTo(memberIt->getName());
            cout << "\n✓ Task assigned to " << memberIt->getName();
        } else {
            cout << "\n✗ Task or team member not found!";
        }
    }
    
    void updateTaskStatus(int taskId, TaskStatus newStatus) {
        auto it = find_if(tasks.begin(), tasks.end(),
                         [taskId](const Task& t) { return t.getTaskId() == taskId; });
        if (it != tasks.end()) {
            it->setStatus(newStatus);
            cout << "\n✓ Task status updated!";
        } else {
            cout << "\n✗ Task not found!";
        }
    }
    
    // Display methods
    void displayProjectInfo() const {
        cout << "\n" << string(60, '=');
        cout << "\nProject ID: " << projectId << " | Name: " << projectName;
        cout << "\nDescription: " << description;
        cout << "\nStatus: ";
        
        switch(status) {
            case NOT_STARTED: cout << "NOT STARTED"; break;
            case PLANNING: cout << "PLANNING"; break;
            case ACTIVE: cout << "ACTIVE"; break;
            case ON_HOLD: cout << "ON HOLD"; break;
            case COMPLETED_P: cout << "COMPLETED"; break;
        }
        
        cout << "\nStart Date: " << startDate << " | End Date: " << endDate;
        cout << "\nTeam Size: " << team.size() << " | Total Tasks: " << tasks.size();
        cout << "\n" << string(60, '=');
    }
    
    void displayTeam() const {
        cout << "\n" << string(60, '=');
        cout << "\nTEAM MEMBERS:";
        if (team.empty()) {
            cout << "\nNo team members assigned yet.";
        } else {
            for (const auto& member : team) {
                member.display();
            }
        }
        cout << "\n" << string(60, '=');
    }
    
    void displayTasks() const {
        cout << "\n" << string(60, '=');
        cout << "\nPROJECT TASKS (" << tasks.size() << " total):";
        if (tasks.empty()) {
            cout << "\nNo tasks created yet.";
        } else {
            for (const auto& task : tasks) {
                task.display();
            }
        }
        cout << "\n" << string(60, '=');
    }                                                                                                              
    
    // Statistics
    void displayStatistics() const {
        cout << "\n" << string(60, '=');
        cout << "\nPROJECT STATISTICS:";
        
        int completed = 0, inProgress = 0, pending = 0, blocked = 0;
        int totalEstimated = 0, totalActual = 0;
        
        for (const auto& task : tasks) {
            totalEstimated += task.getEstimatedHours();
            totalActual += task.getActualHours();
            
            switch(task.getStatus()) {
                case COMPLETED: completed++; break;
                case IN_PROGRESS: inProgress++; break;
                case IN_REVIEW: inProgress++; break;
                case PENDING: pending++; break;
                case BLOCKED: blocked++; break;
            }
        }
        
        cout << "\nTask Breakdown:";
        cout << "\n  Completed: " << completed;
        cout << "\n  In Progress: " << inProgress;
        cout << "\n  Pending: " << pending;
        cout << "\n  Blocked: " << blocked;
        cout << "\n\nTime Tracking:";
        cout << "\n  Total Estimated Hours: " << totalEstimated;
        cout << "\n  Total Actual Hours: " << totalActual;
        
        if (totalEstimated > 0) {
            cout << "\n  Variance: " << (totalActual - totalEstimated) << " hours";
            double variance_percent = ((totalActual - totalEstimated) / (double)totalEstimated) * 100;
            cout << " (" << fixed << setprecision(1) << variance_percent << "%)";
        }
        
        cout << "\n" << string(60, '=');
    }
};

// Portfolio Manager - manages multiple projects
class ProjectPortfolio {
private:
    vector<Project> projects;
    int projectIdCounter;
    
public:
    ProjectPortfolio() : projectIdCounter(100) {}
    
    void createProject(string name, string description, string startDate, string endDate) {
        projectIdCounter++;
        projects.push_back(Project(projectIdCounter, name, description, startDate, endDate));
        cout << "\n✓ Project created with ID: " << projectIdCounter;
    }
    
    Project* findProject(int projectId) {
        auto it = find_if(projects.begin(), projects.end(),
                         [projectId](const Project& p) { return p.getProjectId() == projectId; });
        if (it != projects.end()) {
            return &(*it);
        }
        return nullptr;
    }
    
    void displayAllProjects() const {
        cout << "\n" << string(60, '=');
        cout << "\nALL PROJECTS:";
        if (projects.empty()) {
            cout << "\nNo projects created yet.";
        } else {
            for (const auto& project : projects) {
                project.displayProjectInfo();
            }
        }
        cout << "\n" << string(60, '=');
    }
    
    void displayPortfolioSummary() const {
        cout << "\n" << string(60, '=');
        cout << "\nPORTFOLIO SUMMARY:";
        cout << "\nTotal Projects: " << projects.size();
        
        int active = 0, completed = 0, onHold = 0;
        int totalTeamMembers = 0, totalTasks = 0;
        
        for (const auto& project : projects) {
            if (project.getStatus() == ACTIVE) active++;
            else if (project.getStatus() == COMPLETED_P) completed++;
            else if (project.getStatus() == ON_HOLD) onHold++;
            
            totalTeamMembers += project.getTeamSize();
            totalTasks += project.getTaskCount();
        }
        
        cout << "\nProject Status Breakdown:";
        cout << "\n  Active Projects: " << active;
        cout << "\n  Completed Projects: " << completed;
        cout << "\n  On Hold: " << onHold;
        cout << "\nOrganization Overview:";
        cout << "\n  Total Team Members: " << totalTeamMembers;
        cout << "\n  Total Tasks: " << totalTasks;
        cout << "\n" << string(60, '=');
    }
};

// Main menu and interface
void displayMainMenu() {
    cout << "\n" << string(60, '=');
    cout << "\n--- PROJECT WORK MANAGEMENT SYSTEM ---";
    cout << "\n" << string(60, '=');
    cout << "\n1. Create New Project";
    cout << "\n2. View All Projects";
    cout << "\n3. View Portfolio Summary";
    cout << "\n4. Select Project to Manage";
    cout << "\n5. Exit";
    cout << "\nEnter your choice: ";
}

void displayProjectMenu() {
    cout << "\n" << string(60, '=');
    cout << "\n--- PROJECT MANAGEMENT MENU ---";
    cout << "\n" << string(60, '=');
    cout << "\n1. View Project Info";
    cout << "\n2. Add Team Member";
    cout << "\n3. View Team";
    cout << "\n4. Create Task";
    cout << "\n5. View Tasks";
    cout << "\n6. Assign Task";
    cout << "\n7. Update Task Status";
    cout << "\n8. View Statistics";
    cout << "\n9. Change Project Status";
    cout << "\n10. Back to Main Menu";
    cout << "\nEnter your choice: ";
}

int main() {
    ProjectPortfolio portfolio;
    int choice;
    
    cout << "\n*** PROJECT WORK MANAGEMENT SYSTEM FOR SOFTWARE ENGINEERING ***\n";
    
    while (true) {
        displayMainMenu();
        cin >> choice;
        cin.ignore();
        
        switch (choice) {
            case 1: {
                string name, description, startDate, endDate;
                cout << "\nEnter Project Name: ";
                getline(cin, name);
                cout << "Enter project description: ";
                getline(cin, description);
                cout << "Enter Start Date (YYYY-MM-DD): ";
                getline(cin, startDate);
                cout << "Enter End Date (YYYY-MM-DD): ";
                getline(cin, endDate);
                portfolio.createProject(name, description, startDate, endDate);
                break;
            }
            
            case 2: {
                portfolio.displayAllProjects();
                break;
            }
            
            case 3: {
                portfolio.displayPortfolioSummary();
                break;
            }
            
            case 4: {
                int projectId;
                cout << "\nEnter Project ID: ";
                cin >> projectId;
                cin.ignore();
                
                Project* project = portfolio.findProject(projectId);
                if (project != nullptr) {
                    int projChoice;
                    while (true) {
                        displayProjectMenu();
                        cin >> projChoice;
                        cin.ignore();
                        
                        switch (projChoice) {
                            case 1:
                                project->displayProjectInfo();
                                break;
                                
                            case 2: {
                                string memberId, memberName, role;
                                cout << "\nEnter Member ID: ";
                                getline(cin, memberId);
                                cout << "Enter Member Name: ";
                                getline(cin, memberName);
                                cout << "Enter Role (Developer/Manager/Designer/etc): ";
                                getline(cin, role);
                                project->addTeamMember(memberId, memberName, role);
                                break;
                            }
                            
                            case 3:
                                project->displayTeam();
                                break;
                                
                            case 4: {
                                string title, description, dueDate;
                                int priority, hours;
                                cout << "\nEnter Task Title: ";
                                getline(cin, title);
                                cout << "Enter Description: ";
                                getline(cin, description);
                                cout << "Enter Priority (1=LOW, 2=MEDIUM, 3=HIGH, 4=CRITICAL): ";
                                cin >> priority;
                                cin.ignore();
                                cout << "Enter Due Date (YYYY-MM-DD): ";
                                getline(cin, dueDate);
                                cout << "Enter Estimated Hours: ";
                                cin >> hours;
                                cin.ignore();
                                
                                project->createTask(title, description, (Priority)priority, dueDate, hours);
                                break;
                            }
                            
                            case 5:
                                project->displayTasks();
                                break;
                                
                            case 6: {
                                int taskId;
                                string memberId;
                                cout << "\nEnter Task ID: ";
                                cin >> taskId;
                                cin.ignore();
                                cout << "Enter Member ID: ";
                                getline(cin, memberId);
                                project->assignTask(taskId, memberId);
                                break;
                            }
                            
                            case 7: {
                                int taskId, newStatus;
                                cout << "\nEnter Task ID: ";
                                cin >> taskId;
                                cout << "Enter New Status (0=PENDING, 1=IN_PROGRESS, 2=IN_REVIEW, 3=COMPLETED, 4=BLOCKED): ";
                                cin >> newStatus;
                                cin.ignore();
                                project->updateTaskStatus(taskId, (TaskStatus)newStatus);
                                break;
                            }
                            
                            case 8:
                                project->displayStatistics();
                                break;
                                
                            case 9: {
                                int status;
                                cout << "\nEnter New Status (0=NOT_STARTED, 1=PLANNING, 2=ACTIVE, 3=ON_HOLD, 4=COMPLETED): ";
                                cin >> status;
                                cin.ignore();
                                project->setStatus((ProjectStatus)status);
                                cout << "\n✓ Project status updated!";
                                break;
                            }
                            
                            case 10:
                                goto mainMenu;
                                
                            default:
                                cout << "\nInvalid choice!";
                        }
                    }
                } else {
                    cout << "\n✗ Project not found!";
                }
                mainMenu:
                break;
            }
            
            case 5: {
                cout << "\n*** Thank you for using Project Work Management System ***\n";
                return 0;
            }
            
            default:
                cout << "\nInvalid choice!";
        }
    }
    
    return 0;
}

to Krishna 
