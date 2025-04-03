#include <stdio.h>
#include <string.h>

#define MAX_ROWS 100
#define FILE_NAME "patients.csv"

struct Patient {
    int id;
    char name[50];
    char doctor[50];
    int age;
    float billAmount;
};

int searchPatient(struct Patient patients[], int count);
int updatePatient(struct Patient patients[], int count);
int deletePatient(struct Patient patients[], int *count);
int addPatient(struct Patient patients[], int *count);
int sortPatients(struct Patient patients[], int count);
int displayAllPatients(struct Patient patients[], int count);
int exportPatients(struct Patient patients[], int count);
int importPatients(struct Patient patients[], int *count);

int searchPatient(struct Patient patients[], int count) {
    int searchID;
    char searchName[50];
    printf("Search by:\n1. ID\n2. Name\nEnter choice: ");
    int choice;
    scanf("%d", &choice);

    if (choice == 1) 
    {
        printf("Enter Patient ID: ");
        scanf("%d", &searchID);
    } 
    else if (choice == 2) 
    {
        printf("Enter Patient Name: ");
        scanf(" %[^\n]", searchName);
    }
    else 
    {
        printf("Invalid choice!\n");
        return 0;
    }

    for (int i = 0; i < count; i++) {
        if ((choice == 1 && patients[i].id == searchID) ||
            (choice == 2 && strcmp(patients[i].name, searchName) == 0)) {
            printf("\nPatient Found: ID: %d, Name: %s, Doctor: %s, Age: %d, Bill: %.2f\n",
                   patients[i].id, patients[i].name, patients[i].doctor,
                   patients[i].age, patients[i].billAmount);
            return 1;
        }
    }
    printf("\nPatient not found!\n");
    return 0;
}

int updatePatient(struct Patient patients[], int count) {
    int searchID;
    printf("Enter Patient ID to update: ");
    scanf("%d", &searchID);

    for (int i = 0; i < count; i++) {
        if (patients[i].id == searchID) {
            printf("Enter new Doctor Name: ");
            scanf(" %[^\n]", patients[i].doctor);
            printf("Enter new Age: ");
            scanf("%d", &patients[i].age);
            printf("Enter new Bill Amount: ");
            scanf("%f", &patients[i].billAmount);
            printf("Patient record updated successfully!\n");
            exportPatients(patients, count); 
            return 1;
        }
    }
    printf("Patient not found!\n");
    return 0;
}

int deletePatient(struct Patient patients[], int *count) {
    int searchID, i, found = 0;
    printf("Enter Patient ID to delete: ");
    scanf("%d", &searchID);

    // Find and remove the patient
    for (i = 0; i < *count; i++) {
        if (patients[i].id == searchID) {
            found = 1;
            patients[i] = patients[--(*count)]; // Replace with last patient and reduce count
            break;
        }
    }

    if (!found) {
        printf("Patient not found!\n");
        return 0;
    }

    // Update the file with the new patient list
    FILE *file = fopen(FILE_NAME, "w");
    if (!file) {
        printf("Error opening file for writing!\n");
        return 0;
    }

    for (i = 0; i < *count; i++) {
        fprintf(file, "%d,%s,%s,%d,%.2f\n",
                patients[i].id, patients[i].name, patients[i].doctor,
                patients[i].age, patients[i].billAmount);
    }

    fclose(file);
    printf("Patient record deleted successfully!\n");

    return 1;
}
         

int addPatient(struct Patient patients[], int *count) {
    if (*count >= MAX_ROWS) 
    {
        printf("Cannot add more patients, storage full!\n");
        return 0;
    }

    printf("Enter Patient ID: ");
    scanf("%d", &patients[*count].id);
    printf("Enter Name: ");
    scanf(" %[^\n]", patients[*count].name);
    printf("Enter Doctor Name: ");
    scanf(" %[^\n]", patients[*count].doctor);
    printf("Enter Age: ");
    scanf("%d", &patients[*count].age);
    printf("Enter Bill Amount: ");
    scanf("%f", &patients[*count].billAmount);

    (*count)++;
    printf("Patient record added successfully!\n");

    exportPatients(patients, *count); 
    return 1;
}
int sortPatients(struct Patient patients[], int count) {
    int choice;
    printf("Sort by:\n1. Name\n2. Age\n3. Bill Amount\nEnter choice: ");
    scanf("%d", &choice);

    for (int i = 0; i < count - 1; i++) {
        for (int j = 0; j < count - i - 1; j++) { 
            int swap = 0;

            if ((choice == 1 && strcmp(patients[j].name, patients[j + 1].name) > 0) ||
                (choice == 2 && patients[j].age > patients[j + 1].age) ||
                (choice == 3 && patients[j].billAmount > patients[j + 1].billAmount)) {
                swap = 1;
            }

            if (swap) {
                struct Patient temp = patients[j];
                patients[j] = patients[j + 1];
                patients[j + 1] = temp;
            }
        }
    }

    printf("Patients sorted successfully!\n");
    return 1;
}

int exportPatients(struct Patient patients[], int count) {
    FILE *file = fopen(FILE_NAME, "w"); 
    if (!file)
    {
        printf("Error opening file for writing!\n");
        return 0;
    }

    for (int i = 0; i < count; i++) {
        fprintf(file, "%d,%s,%s,%d,%.2f\n",
                patients[i].id, patients[i].name, patients[i].doctor,
                patients[i].age, patients[i].billAmount);
    }

    fclose(file);
    printf("Patients saved successfully!\n");
    return 1;
}

int importPatients(struct Patient patients[], int *count) {
    FILE *file = fopen(FILE_NAME, "r");
    if (!file)
    {
        printf("No existing file found. Starting fresh.\n");
        return 0;
    }

    *count = 0;
    while (fscanf(file, "%d,%49[^,],%49[^,],%d,%f\n",
                  &patients[*count].id,
                  patients[*count].name,
                  patients[*count].doctor,
                  &patients[*count].age,
                  &patients[*count].billAmount) == 5) {
        (*count)++;
    }

    fclose(file);
    printf("Patients imported successfully! (%d records loaded)\n", *count);
    return 1;
}

int displayAllPatients(struct Patient patients[], int count) {
    if (count == 0)
    {
        printf("\nNo patients in the record!\n");
        return 0;
    }

    printf("\n%-5s %-20s %-20s %-5s %-10s\n", "ID", "Name", "Doctor", "Age", "Bill");
    printf("--------------------------------------------------------------\n");

    for (int i = 0; i < count; i++) 
    {
        printf("%-5d %-20s %-20s %-5d %-10.2f\n",
               patients[i].id, patients[i].name, patients[i].doctor,
               patients[i].age, patients[i].billAmount);
    }
    return 1;
}

int main() {
    struct Patient patients[MAX_ROWS];
    int count = 0, choice;

    importPatients(patients, &count);

    while (1)
    {
        printf("\nMenu:\n1. Search\n2. Update\n3. Delete\n4. Add New Patient\n5. Sort\n6. Display All Patients\n7. Exit\nEnter choice: ");
        scanf("%d", &choice);

        if (choice == 1)
        {
             searchPatient(patients, count);
        }
        else if (choice == 2) 
        {
            updatePatient(patients, count);
        }
        else if (choice == 3) 
        {
            deletePatient(patients, &count);
        }
        else if (choice == 4) 
        {
            addPatient(patients, &count);
        }
        else if (choice == 5) 
        {
            sortPatients(patients, count);
        }
        else if (choice == 6) 
        {
            displayAllPatients(patients, count);
        }
        else if (choice == 7) 
        {
            break;
        }
        else 
        {
            printf("Invalid choice! Try again.\n");
        }
    }

    return 0;
}
