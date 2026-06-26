#include <stdio.h>
#include <string.h>

int main() {
    int masterChoice;
    int ch; 

    // File pointers used for storage
    FILE *file, *tempFile;

    // --- Admin Variables ---
    char adminUsername[20], adminPassword[20];
    int memberID = 0;
    char memberName[50] = "";

    // --- Librarian Variables ---
    char libUsername[20], libPassword[20];
    int bookID = 0;
    char bookTitle[50] = "";

    // --- Student Variables ---
    char regName[50] = "", regPass[50] = "", regDept[50] = "";
    int regAge = 0, regID = 0;
    
    char loginName[50], loginPass[50];
    char searchBook[50];
    char libraryBook[50] = "harry potters"; 
    
    int isLoggedIn = 0;   
    int isIssued = 0;     
    int isFinePaid = 1;   
    int choice;           

    // Load dynamic student state from file on startup (keeps track of books/fines)
    file = fopen("student_state.txt", "r");
    if (file != NULL) {
        fscanf(file, "%d,%d", &isIssued, &isFinePaid);
        fclose(file);
    }

    while (1) {
        printf("\n--- LIBRARY MANAGEMENT SYSTEM ---\n");
        printf("1. Admin Login\n");
        printf("2. Librarian Login\n");
        printf("3. Student Portal\n");
        printf("4. Exit System\n");
        printf("Enter your choice (1-4): ");
        
        if (scanf("%d", &masterChoice) != 1) {
            printf("Invalid input. Clearing buffer...\n");
            while ((ch = getchar()) != '\n' && ch != EOF);
            continue;
        }

        // ---------------------------------------------------------------------
        // PORTAL 1: ADMIN SYSTEM (Manages members.txt)
        // ---------------------------------------------------------------------
        if (masterChoice == 1) {
            printf("\n--- ADMIN LOGIN ---\n");
            printf("Enter Username: ");
            scanf("%19s", adminUsername);
            printf("Enter Password: ");
            scanf("%19s", adminPassword);

            if (strcmp(adminUsername, "admin") != 0 || strcmp(adminPassword, "123") != 0) {
                printf("Login failed! Wrong credentials.\n");
            } else {
                printf("Login successful!\n");
                while (1) {
                    printf("\n--- ADMIN MENU ---\n");
                    printf("3. Add Member\n");
                    printf("4. Delete Member\n");
                    printf("5. Search Member\n");
                    printf("6. Edit Member\n");
                    printf("7. View Member\n");
                    printf("8. Logout\n");
                    printf("Enter choice (3-8): ");
                    
                    while ((ch = getchar()) != '\n' && ch != EOF); 
                    if (scanf("%d", &choice) != 1) continue;

                    if (choice == 3) {
                        printf("Enter Member ID: ");
                        scanf("%d", &memberID);
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter Member Name: ");
                        scanf("%49[^\n]", memberName); 

                        // Open members file and append new record at the bottom
                        file = fopen("members.txt", "a");
                        if (file != NULL) {
                            fprintf(file, "%d,%s\n", memberID, memberName);
                            fclose(file);
                            printf("Member added successfully!\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 4) {
                        int deleteID;
                        int found = 0;
                        printf("Enter Member ID to delete: ");
                        scanf("%d", &deleteID);

                        // Twin-File Trick: Copy everything EXCEPT the deleted ID to temp.txt
                        file = fopen("members.txt", "r");
                        tempFile = fopen("temp.txt", "w");
                        
                        if (file != NULL && tempFile != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &memberID, memberName) != EOF) {
                                if (memberID == deleteID) {
                                    found = 1; 
                                } else {
                                    fprintf(tempFile, "%d,%s\n", memberID, memberName);
                                }
                            }
                            fclose(file);
                            fclose(tempFile);
                            remove("members.txt");
                            rename("temp.txt", "members.txt");
                            
                            if (found) printf("Member deleted successfully!\n");
                            else printf("Member ID not found.\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 5) {
                        int searchID, found = 0;
                        printf("Enter Member ID to search: ");
                        scanf("%d", &searchID);
                        
                        file = fopen("members.txt", "r");
                        if (file != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &memberID, memberName) != EOF) {
                                if (memberID == searchID) {
                                    printf("Member Found! Name: %s\n", memberName);
                                    found = 1;
                                    break;
                                }
                            }
                            fclose(file);
                            if (!found) printf("Member not found.\n");
                        } else {
                            printf("No members registered yet.\n");
                        }
                    }
                    else if (choice == 6) {
                        int editID, found = 0;
                        char newName[50];
                        printf("Enter Member ID to edit: ");
                        scanf("%d", &editID);
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter New Name: ");
                        scanf("%49[^\n]", newName);

                        // Twin-File Trick: Swap old name for new name during file copying
                        file = fopen("members.txt", "r");
                        tempFile = fopen("temp.txt", "w");

                        if (file != NULL && tempFile != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &memberID, memberName) != EOF) {
                                if (memberID == editID) {
                                    fprintf(tempFile, "%d,%s\n", memberID, newName);
                                    found = 1;
                                } else {
                                    fprintf(tempFile, "%d,%s\n", memberID, memberName);
                                }
                            }
                            fclose(file);
                            fclose(tempFile);
                            remove("members.txt");
                            rename("temp.txt", "members.txt");
                            
                            if (found) printf("Member name updated!\n");
                            else printf("Member ID not found.\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 7) {
                        file = fopen("members.txt", "r");
                        if (file != NULL) {
                            int hasRecords = 0;
                            while (fscanf(file, "%d,%49[^\n]\n", &memberID, memberName) != EOF) {
                                printf("ID: %d, Name: %s\n", memberID, memberName);
                                hasRecords = 1;
                            }
                            fclose(file);
                            if (!hasRecords) printf("The member directory is empty.\n");
                        } else {
                            printf("The member directory is empty.\n");
                        }
                    }
                    else if (choice == 8) {
                        printf("Logging out to main menu.\n");
                        break;
                    }
                    else {
                        printf("Invalid choice. Try again.\n");
                    }
                }
            }
        }

        // ---------------------------------------------------------------------
        // PORTAL 2: LIBRARIAN SYSTEM (Manages books.txt)
        // ---------------------------------------------------------------------
        else if (masterChoice == 2) {
            printf("\n--- LIBRARIAN LOGIN ---\n");
            printf("Enter Username: ");
            scanf("%19s", libUsername);
            printf("Enter Password: ");
            scanf("%19s", libPassword);

            if (strcmp(libUsername, "librarian") != 0 || strcmp(libPassword, "123") != 0) {
                printf("Login failed! Wrong credentials.\n");
            } else {
                printf("Login successful!\n");
                while (1) {
                    printf("\n--- LIBRARIAN MENU ---\n");
                    printf("3. Add Book\n");
                    printf("4. Remove Book\n");
                    printf("5. Search Book\n");
                    printf("6. Edit Book\n");
                    printf("7. View Book\n");
                    printf("8. Logout\n");
                    printf("Enter choice (3-8): ");
                    
                    while ((ch = getchar()) != '\n' && ch != EOF); 
                    if (scanf("%d", &choice) != 1) continue;

                    if (choice == 3) {
                        printf("Enter Book ID: ");
                        scanf("%d", &bookID);
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter Book Title: ");
                        scanf("%49[^\n]", bookTitle); 

                        file = fopen("books.txt", "a");
                        if (file != NULL) {
                            fprintf(file, "%d,%s\n", bookID, bookTitle);
                            fclose(file);
                            printf("Book added successfully!\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 4) {
                        int removeID, found = 0;
                        printf("Enter Book ID to remove: ");
                        scanf("%d", &removeID);

                        file = fopen("books.txt", "r");
                        tempFile = fopen("temp.txt", "w");

                        if (file != NULL && tempFile != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &bookID, bookTitle) != EOF) {
                                if (bookID == removeID) {
                                    found = 1;
                                } else {
                                    fprintf(tempFile, "%d,%s\n", bookID, bookTitle);
                                }
                            }
                            fclose(file);
                            fclose(tempFile);
                            remove("books.txt");
                            rename("temp.txt", "books.txt");

                            if (found) printf("Book removed successfully!\n");
                            else printf("Book not found.\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 5) {
                        int searchID, found = 0;
                        printf("Enter Book ID to search: ");
                        scanf("%d", &searchID);

                        file = fopen("books.txt", "r");
                        if (file != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &bookID, bookTitle) != EOF) {
                                if (bookID == searchID) {
                                    printf("Book Found! ID: %d, Title: %s\n", bookID, bookTitle);
                                    found = 1;
                                    break;
                                }
                            }
                            fclose(file);
                            if (!found) printf("Book not found.\n");
                        } else {
                            printf("The library catalog is empty.\n");
                        }
                    }
                    else if (choice == 6) {
                        int editID, found = 0;
                        char newTitle[50];
                        printf("Enter Book ID to edit: ");
                        scanf("%d", &editID);
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter New Book Title: ");
                        scanf("%49[^\n]", newTitle);

                        file = fopen("books.txt", "r");
                        tempFile = fopen("temp.txt", "w");

                        if (file != NULL && tempFile != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &bookID, bookTitle) != EOF) {
                                if (bookID == editID) {
                                    fprintf(tempFile, "%d,%s\n", bookID, newTitle);
                                    found = 1;
                                } else {
                                    fprintf(tempFile, "%d,%s\n", bookID, bookTitle);
                                }
                            }
                            fclose(file);
                            fclose(tempFile);
                            remove("books.txt");
                            rename("temp.txt", "books.txt");

                            if (found) printf("Book title updated!\n");
                            else printf("Book not found.\n");
                        } else {
                            printf("File error.\n");
                        }
                    }
                    else if (choice == 7) {
                        file = fopen("books.txt", "r");
                        if (file != NULL) {
                            int hasRecords = 0;
                            while (fscanf(file, "%d,%49[^\n]\n", &bookID, bookTitle) != EOF) {
                                printf("ID: %d, Title: %s\n", bookID, bookTitle);
                                hasRecords = 1;
                            }
                            fclose(file);
                            if (!hasRecords) printf("The library catalog is empty.\n");
                        } else {
                            printf("The library catalog is empty.\n");
                        }
                    }
                    else if (choice == 8) {
                        printf("Logging out to main menu.\n");
                        break;
                    }
                    else {
                        printf("Invalid choice. Try again.\n");
                    }
                }
            }
        }

        // ---------------------------------------------------------------------
        // PORTAL 3: STUDENT PORTAL (Manages students.txt & student_state.txt)
        // ---------------------------------------------------------------------
        else if (masterChoice == 3) {
            int exitStudentPortal = 0;
            
            while (exitStudentPortal == 0) {
                if (isLoggedIn == 0) {
                    printf("\n--- STUDENT PORTAL ---\n");
                    printf("1. Sign Up\n");
                    printf("2. Login\n");
                    printf("6. Back to Main Menu\n");
                    printf("Enter choice: ");
                    
                    while ((ch = getchar()) != '\n' && ch != EOF); 
                    if (scanf("%d", &choice) != 1) continue;

                    if (choice == 1) {
                        printf("\n--- SIGN UP ---\n");
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter Name: ");
                        scanf("%49[^\n]", regName); 
                        printf("Enter Age: ");
                        scanf("%d", &regAge);
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter Department: ");
                        scanf("%49[^\n]", regDept); 
                        printf("Enter Student ID: ");
                        scanf("%d", &regID);
                        
                        while ((ch = getchar()) != '\n' && ch != EOF);
                        printf("Enter Password: ");
                        scanf("%49s", regPass);
                        
                        // Overwrites students.txt with the new single profile setup
                        file = fopen("students.txt", "w"); 
                        if (file != NULL) {
                            fprintf(file, "%s,%d,%s,%d,%s\n", regName, regAge, regDept, regID, regPass);
                            fclose(file);
                            printf("Account created successfully!\n");
                        } else {
                            printf("Error saving profile.\n");
                        }
                    }
                    else if (choice == 2) {
                        file = fopen("students.txt", "r");
                        if (file == NULL) {
                            printf("No account found. Please sign up first.\n");
                        } else {
                            // Fetch account details directly out of file storage
                            fscanf(file, "%49[^,],%d,%49[^,],%d,%49s\n", regName, &regAge, regDept, &regID, regPass);
                            fclose(file);

                            printf("\n--- LOGIN ---\n");
                            while ((ch = getchar()) != '\n' && ch != EOF);
                            printf("Enter Name: ");
                            scanf("%49[^\n]", loginName); 
                            printf("Enter Password: ");
                            scanf("%49s", loginPass);

                            if (strcmp(loginName, regName) == 0 && strcmp(loginPass, regPass) == 0) {
                                isLoggedIn = 1;
                                printf("Welcome, %s! Login successful.\n", regName);
                            } else {
                                printf("Wrong username or password.\n");
                            }
                        }
                    }
                    else if (choice == 6) {
                        exitStudentPortal = 1; 
                    }
                    else {
                        printf("Invalid choice.\n");
                    }
                }
                
                else {
                    printf("\n--- STUDENT DASHBOARD ---\n");
                    printf("2. Search Book\n");
                    printf("3. Check Availability & Request Issue\n");
                    printf("4. View Issued Book Timeline\n");
                    printf("5. Check Fine Status\n");
                    printf("6. Logout\n");
                    printf("Enter choice (2-6): ");

                    while ((ch = getchar()) != '\n' && ch != EOF); 
                    if (scanf("%d", &choice) != 1) continue;

                    if (choice == 2) {
                        while ((ch = getchar()) != '\n' && ch != EOF); 
                        
                        printf("\nEnter Book name to search: ");
                        scanf("%49[^\n]", searchBook); 
                        
                        // Scans the active librarian book file records
                        int bookFoundInCatalog = 0;
                        file = fopen("books.txt", "r");
                        if (file != NULL) {
                            while (fscanf(file, "%d,%49[^\n]\n", &bookID, bookTitle) != EOF) {
                                if (strcasecmp(searchBook, bookTitle) == 0) {
                                    strcpy(libraryBook, bookTitle);
                                    bookFoundInCatalog = 1;
                                    break;
                                }
                            }
                            fclose(file);
                        }

                        // Code fallback match if file catalog is empty
                        if (!bookFoundInCatalog && strcasecmp(searchBook, "harry potters") == 0) {
                            strcpy(libraryBook, "harry potters");
                            bookFoundInCatalog = 1;
                        }

                        if (bookFoundInCatalog) {
                            printf("Book found! Title: %s\n", libraryBook);
                            printf("Proceeding to issue the book...\n");
                            if (isIssued == 1) {
                                printf("You already have an issued book.\n");
                            } else {
                                printf("Book issued to you successfully!\n");
                                isIssued = 1;
                            }
                        } else {
                            printf("Book not found.\n");
                        }
                    }
                    else if (choice == 3) {
                        if (isIssued == 1) {
                            printf("You already have an issued book.\n");
                        } else {
                            printf("\nAvailable Book: %s\n", libraryBook);
                            printf("Book issued to you successfully!\n");
                            isIssued = 1;
                        }
                    }
                    else if (choice == 4) {
                        if (isIssued == 1) {
                            printf("\n--- BOOK TIMELINE ---\n");
                            printf("Book Title: %s\n", libraryBook);
                            printf("Return Rule: Return within 14 days.\n");
                        } else {
                            printf("You do not have any issued books.\n");
                        }
                    }
                    else if (choice == 5) {
                        printf("\n--- FINE STATUS ---\n");
                        if (isFinePaid == 1) {
                            printf("Status: PAID (No fines).\n");
                            printf("Changing status to UNPAID for testing.\n");
                            isFinePaid = 0; 
                        } else {
                            printf("Status: NOT PAID (Fine pending).\n");
                            printf("Processing automatic payment... Fine paid!\n");
                            isFinePaid = 1; 
                        }
                    }
                    else if (choice == 6) {
                        isLoggedIn = 0;
                        printf("Logged out successfully.\n");
                    }
                    else {
                        printf("Invalid choice.\n");
                    }

                    // Save the active state of fines/issues to file storage
                    file = fopen("student_state.txt", "w");
                    if (file != NULL) {
                        fprintf(file, "%d,%d", isIssued, isFinePaid);
                        fclose(file);
                    }
                }
            }
        }

        // ---------------------------------------------------------------------
        // PORTAL 4: SYSTEM SHUTDOWN
        // ---------------------------------------------------------------------
        else if (masterChoice == 4) {
            printf("\nShutting down the library system. Goodbye!\n");
            break;
        }
        else {
            printf("Invalid choice. Please select between 1 and 4.\n");
        }
    }

    return 0;
}
