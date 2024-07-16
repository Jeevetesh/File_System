#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

#define MAX_FILES 100
#define MAX_FILENAME 20
#define MAX_FILESIZE 1024
#define FS_DIRECTORY "./filesystem/"
#define FS_STATE_FILE "./filesystem_state.txt"

typedef struct {
    char name[MAX_FILENAME];
} Inode;

typedef struct {
    Inode inodes[MAX_FILES];
} FileSystem;

FileSystem fs;

void initFileSystem() {
    memset(&fs, 0, sizeof(FileSystem));
    mkdir(FS_DIRECTORY, 0777);  // Create a directory to store files
}

void saveFileSystemState() {
    FILE *file = fopen(FS_STATE_FILE, "wb");
    if (file == NULL) {
        printf("Error: Could not save file system state.\n");
        return;
    }
    fwrite(&fs, sizeof(FileSystem), 1, file);
    fclose(file);
}

void loadFileSystemState() {
    FILE *file = fopen(FS_STATE_FILE, "rb");
    if (file != NULL) {
        fread(&fs, sizeof(FileSystem), 1, file);
        fclose(file);
    }
}

int createFile(const char *name) {
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    for (int i = 0; i < MAX_FILES; i++) {
        if (fs.inodes[i].name[0] == '\0') {
            strncpy(fs.inodes[i].name, name, MAX_FILENAME);
            snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, name);
            FILE *file = fopen(filePath, "w");
            if (file == NULL) {
                printf("Error: Could not create file on disk.\n");
                memset(&fs.inodes[i], 0, sizeof(Inode));
                return -1;
            }
            fclose(file);
            saveFileSystemState();
            return i;
        }
    }
    return -1;
}

int writeFile(int fileIndex, const char *data) {
    if (fileIndex < 0 || fileIndex >= MAX_FILES) return -1;
    Inode *inode = &fs.inodes[fileIndex];
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, inode->name);
    FILE *file = fopen(filePath, "w");
    if (file == NULL) return -1;
    fwrite(data, sizeof(char), strlen(data), file);
    fclose(file);
    return 0;
}

int readFile(int fileIndex, char *buffer) {
    if (fileIndex < 0 || fileIndex >= MAX_FILES) return -1;
    Inode *inode = &fs.inodes[fileIndex];
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, inode->name);
    FILE *file = fopen(filePath, "r");
    if (file == NULL) return -1;
    size_t bytesRead = fread(buffer, sizeof(char), MAX_FILESIZE, file);
    buffer[bytesRead] = '\0';
    fclose(file);
    return bytesRead;
}

int deleteFile(int fileIndex) {
    if (fileIndex < 0 || fileIndex >= MAX_FILES) return -1;
    Inode *inode = &fs.inodes[fileIndex];
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, inode->name);
    if (remove(filePath) != 0) return -1;
    memset(&fs.inodes[fileIndex], 0, sizeof(Inode));
    saveFileSystemState();
    return 0;
}

void listFiles() {
    printf("Files:\n");
    for (int i = 0; i < MAX_FILES; i++) {
        if (fs.inodes[i].name[0] != '\0') {
            printf("Name: %s\n", fs.inodes[i].name);
        }
    }
}

int truncateFile(int fileIndex) {
    if (fileIndex < 0 || fileIndex >= MAX_FILES) return -1;
    Inode *inode = &fs.inodes[fileIndex];
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, inode->name);
    FILE *file = fopen(filePath, "w"); // Open the file in write mode to truncate it
    if (file == NULL) return -1;
    fclose(file);
    return 0;
}

int modifyFile(int fileIndex, const char *data) {
    if (fileIndex < 0 || fileIndex >= MAX_FILES) return -1;
    Inode *inode = &fs.inodes[fileIndex];
    char filePath[MAX_FILENAME + sizeof(FS_DIRECTORY)];
    snprintf(filePath, sizeof(filePath), "%s%s", FS_DIRECTORY, inode->name);
    FILE *file = fopen(filePath, "a"); // Open the file in append mode to modify it
    if (file == NULL) return -1;
    fwrite(data, sizeof(char), strlen(data), file);
    fclose(file);
    return 0;
}

void menu() {
    int choice;
    char name[MAX_FILENAME];
    char data[MAX_FILESIZE];
    char buffer[MAX_FILESIZE];
    int fileIndex;

    while (1) {
        printf("\n--- File System Menu ---\n");
        printf("1. Create File\n");
        printf("2. Write to File\n");
        printf("3. Read from File\n");
        printf("4. Delete File\n");
        printf("5. List Files\n");
        printf("6. Truncate File\n");
        printf("7. Modify File\n");
        printf("8. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                printf("Enter file name: ");
                scanf("%s", name);
                fileIndex = createFile(name);
                if (fileIndex == -1) {
                    printf("Error: Could not create file. Maximum number of files reached.\n");
                } else {
                    printf("File created with index %d.\n", fileIndex);
                }
                break;
            case 2:
                printf("Enter file index: ");
                scanf("%d", &fileIndex);
                printf("Enter data to write: ");
                scanf(" %[^\n]", data);
                if (writeFile(fileIndex, data) == -1) {
                    printf("Error: Could not write to file.\n");
                } else {
                    printf("Data written to file.\n");
                }
                break;
            case 3:
                printf("Enter file index: ");
                scanf("%d", &fileIndex);
                if (readFile(fileIndex, buffer) == -1) {
                    printf("Error: Could not read from file.\n");
                } else {
                    printf("Data read from file: %s\n", buffer);
                }
                break;
            case 4:
                printf("Enter file index: ");
                scanf("%d", &fileIndex);
                if (deleteFile(fileIndex) == -1) {
                    printf("Error: Could not delete file.\n");
                } else {
                    printf("File deleted.\n");
                }
                break;
            case 5:
                listFiles();
                break;
            case 6:
                printf("Enter file index: ");
                scanf("%d", &fileIndex);
                if (truncateFile(fileIndex) == -1) {
                    printf("Error: Could not truncate file.\n");
                } else {
                    printf("File truncated.\n");
                }
                break;
            case 7:
                printf("Enter file index: ");
                scanf("%d", &fileIndex);
                printf("Enter data to modify: ");
                scanf(" %[^\n]", data);
                if (modifyFile(fileIndex, data) == -1) {
                    printf("Error: Could not modify file.\n");
                } else {
                    printf("File modified.\n");
                }
                break;
            case 8:
                saveFileSystemState();
                exit(0);
            default:
                printf("Invalid choice. Please try again.\n");
        }
    }
}

int main() {
    initFileSystem();
    loadFileSystemState();
    menu();
    return 0;
}
