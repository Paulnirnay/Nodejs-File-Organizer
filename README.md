# Node.js File Organizer

A simple command-line utility that organizes files inside a chosen folder by their file extensions. It automatically creates a separate folder for each extension and moves matching files into it.

## Features

- Organizes files by extension (`.jpg`, `.png`, `.mp3`, `.pdf`, and so on).
- Creates destination folders automatically, such as `Jpg Folder`, `Png Folder`, and `Mp3 Folder`.
- Treats extensions case-insensitively (`.JPG` and `.jpg` go into the same folder).
- Reuses destination folders if they already exist.
- Skips existing directories instead of processing their contents.
- Places files without extensions in `No Extension Folder`.
- Avoids intentionally overwriting a destination file with the same name.
- Uses only built-in Node.js modules; no npm packages are required.

## Requirements

- [Node.js](https://nodejs.org/) installed on your computer.

## Getting Started

1. Save the program as `organizer.js`.
2. Open a terminal in the directory containing `organizer.js`.
3. Run:

   ```bash
   node organizer.js
   ```

4. When prompted, enter the path of the folder you want to organize.

Example on Windows:

```text
C:\Users\YourName\Downloads
```

Example on Linux or macOS:

```text
/home/yourname/Downloads
```

You can also paste a path surrounded by quotation marks.

## How It Works

1. The program asks for a target folder path.
2. It cleans up the input and resolves it to an absolute path.
3. It checks that the path exists and points to a directory.
4. It reads the entries in the target folder.
5. For each regular file, it extracts the extension.
6. It builds a destination folder name from the extension.
7. It creates the destination folder if needed.
8. It checks whether a file with the same name already exists at the destination.
9. If no destination conflict is found, it moves the file and reports the result.
10. It prints a summary of how many files were moved.

## Example

### Before

```text
My Files/
├── file1.jpg
├── file2.png
├── file3.mp3
├── file4.wav
├── file5.png
└── file6.jpg
```

### After

```text
My Files/
├── Jpg Folder/
│   ├── file1.jpg
│   └── file6.jpg
├── Png Folder/
│   ├── file2.png
│   └── file5.png
├── Mp3 Folder/
│   └── file3.mp3
└── Wav Folder/
    └── file4.wav
```

## Source Code

Save the following code in `organizer.js`:

```javascript
const fs = require("fs");
const path = require("path");
const readline = require("readline");

// Ask the user for the folder path
const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.question("Enter the folder path to organize: ", (inputPath) => {
    const targetFolder = path.resolve(inputPath.trim().replace(/^["']|["']$/g, ""));

    // Check whether the folder exists
    if (!fs.existsSync(targetFolder) || !fs.statSync(targetFolder).isDirectory()) {
        console.log("Error: The specified folder does not exist.");
        rl.close();
        return;
    }

    try {
        // Read all items inside the target folder
        const items = fs.readdirSync(targetFolder, {
            withFileTypes: true
        });

        let movedCount = 0;

        for (const item of items) {
            // Skip directories
            if (!item.isFile()) continue;

            const fileName = item.name;
            const filePath = path.join(targetFolder, fileName);

            // Get the file extension
            const extension = path.extname(fileName)
                .slice(1)
                .toLowerCase();

            // Put files without extensions in a separate folder
            const folderName = extension
                ? `${extension.charAt(0).toUpperCase()}${extension.slice(1)} Folder`
                : "No Extension Folder";

            const destinationFolder = path.join(targetFolder, folderName);

            // Create the destination folder if it doesn't exist
            fs.mkdirSync(destinationFolder, { recursive: true });

            const destinationPath = path.join(destinationFolder, fileName);

            // Avoid overwriting files with the same name
            if (fs.existsSync(destinationPath)) {
                console.log(`Skipped: ${fileName} (destination already exists)`);
                continue;
            }

            // Move the file
            fs.renameSync(filePath, destinationPath);

            console.log(`Moved: ${fileName} -> ${folderName}/`);
            movedCount++;
        }

        console.log(`\nOrganization complete! ${movedCount} file(s) moved.`);
    } catch (error) {
        console.error("Error organizing files:", error.message);
    }

    rl.close();
});
```

## Important Notes

- **Files are moved, not copied.** The program changes their location inside the target folder.
- **Only the selected folder is processed.** Files inside existing subfolders are not scanned.
- **Duplicate filenames are skipped.** The program does not automatically rename duplicates.
- **Test it on a sample folder first.** Make sure the result is what you expect before using it on important files.
- The program is intended for ordinary files and folders. Filesystem permissions, locked files, or other filesystem errors can prevent a move.

## Possible Improvements

Ideas for future versions:

- Add a dry-run mode that previews changes without moving files.
- Add an option to organize subfolders recursively.
- Rename conflicting files instead of skipping them.
- Group related extensions into categories such as Images, Videos, Audio, and Documents.
- Add a graphical user interface.
