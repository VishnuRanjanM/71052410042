#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct BookNode {
    char studentID[20];       // changed from int to string
    int bookID;
    char title[100], date[20];
    struct BookNode *next;
} BookNode;

typedef struct Department {
    char name[50];
    BookNode *head;
    struct Department *next;
} Department;

Department *deptList = NULL;

Department* getDepartment(const char *name) {
    Department *cur = deptList;
    while (cur) {
        if (strcmp(cur->name, name) == 0) return cur;
        cur = cur->next;
    }
    Department *newDept = (Department*)malloc(sizeof(Department));
    if (!newDept) {
        printf("Memory allocation failed.\n");
        exit(1);
    }
    strcpy(newDept->name, name);
    newDept->head = NULL;
    newDept->next = deptList;
    deptList = newDept;
    return newDept;
}

void issueBook(BookNode **head, char *sID, int bID, char *title, char *date) {
    BookNode *newNode = (BookNode*)malloc(sizeof(BookNode));
    if (!newNode) {
        printf("Memory allocation failed.\n");
        exit(1);
    }
    strcpy(newNode->studentID, sID);   // store string ID
    newNode->bookID = bID;
    strcpy(newNode->title, title);
    strcpy(newNode->date, date);
    newNode->next = *head;
    *head = newNode;
    printf("Book issued.\n");
}

void returnBook(BookNode **head, int bID) {
    BookNode *cur = *head, *prev = NULL;
    while (cur && cur->bookID != bID) {
        prev = cur;
        cur = cur->next;
    }
    if (!cur) {
        printf("Book not found.\n");
        return;
    }
    if (prev) prev->next = cur->next;
    else *head = cur->next;
    free(cur);
    printf("Book returned.\n");
}

void searchByStudent(BookNode *head, char *sID) {
    int found = 0;
    while (head) {
        if (strcmp(head->studentID, sID) == 0) {   // compare strings
            printf("BookID: %d, Title: %s, Date: %s\n",
                   head->bookID, head->title, head->date);
            found = 1;
        }
        head = head->next;
    }
    if (!found) printf("No books for Student ID %s.\n", sID);
}

void displayHistory(BookNode *head) {
    if (!head) {
        printf("No books borrowed.\n");
        return;
    }
    while (head) {
        printf("StudentID:%s BookID:%d Title:%s Date:%s\n",
               head->studentID, head->bookID, head->title, head->date);
        head = head->next;
    }
}

int countBooks(BookNode *head) {
    int count = 0;
    while (head) {
        count++;
        head = head->next;
    }
    return count;
}

void freeBooks(BookNode *head) {
    while (head) {
        BookNode *temp = head;
        head = head->next;
        free(temp);
    }
}

void freeDepartments() {
    while (deptList) {
        Department *temp = deptList;
        freeBooks(temp->head);
        deptList = deptList->next;
        free(temp);
    }
}

int main() {
    Department *currDept = NULL;
    int choice, bID;
    char sID[20], deptName[50], title[100], date[20];

    while (1) {
        printf("\n1. Switch/Create Dept\n2. Issue\n3. Return\n4. Search by Student\n");
        printf("5. Display History\n6. Count Books\n7. Exit\nChoice: ");
        if (scanf("%d", &choice) != 1) break;

        switch (choice) {
            case 1:
                printf("Dept name: ");
                scanf("%s", deptName);
                currDept = getDepartment(deptName);
                printf("Switched to %s\n", deptName);
                break;
            case 2:
                if (!currDept) { printf("Select department first.\n"); break; }
                printf("Student ID: ");
                scanf("%s", sID);   // read string ID
                printf("Book ID: ");
                scanf("%d", &bID);
                printf("Book Title: ");
                scanf(" %[^\n]%*c", title);
                printf("Date (YYYY-MM-DD): ");
                scanf("%s", date);
                issueBook(&currDept->head, sID, bID, title, date);
                break;
            case 3:
                if (!currDept) { printf("Select department first.\n"); break; }
                printf("Book ID to return: ");
                scanf("%d", &bID);
                returnBook(&currDept->head, bID);
                break;
            case 4:
                if (!currDept) { printf("Select department first.\n"); break; }
                printf("Student ID: ");
                scanf("%s", sID);
                searchByStudent(currDept->head, sID);
                break;
            case 5:
                if (!currDept) { printf("Select department first.\n"); break; }
                displayHistory(currDept->head);
                break;
            case 6:
                if (!currDept) { printf("Select department first.\n"); break; }
                printf("Total books borrowed: %d\n", countBooks(currDept->head));
                break;
            case 7:
                printf("Exiting.\n");
                freeDepartments();
                return 0;
            default:
                printf("Invalid choice.\n");
        }
    }
    freeDepartments();
    return 0;
}
