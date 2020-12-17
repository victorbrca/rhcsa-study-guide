1.i Create hard and soft links
===

**Commands:**
- ln (1)               - make links between files
- unlink (1)           - call the unlink function to remove the specified file

## Definition

### Hardlink

You can think a hard link as an additional name for an existing file. Hard links are associating two or more file names with the same inode . You can create one or more hard links for a single file. Hard links cannot be created for directories and files on a different filesystem or partition.

### Softlink/Symlinks

A soft link is something like a shortcut in Windows. It is an indirect pointer to a file or directory. Unlike a hard link, a symbolic link can point to a file or a directory on a different filesystem or partition.

### Hardlink vs softlink
+ Hardlink  
  + Points to the same inode (2 files with same inode)
  + Cannot be directories
  + Cannot cross filesystems
  + Deleting the original (or hard link) file will not remove the inode (only if both are removed)
+ Softlink  
  + Is just a pointer
  + Deleting a symlink will not delete the file
  + Deleting the original file will not delete the link (but link will be broken)
  + Permission is shown as `lrwxrwxrwx` and reflects the actual file permission

_Deleting a link (hard or soft) does not delete the file_

## Working with Links

### Hardlink

Create a hard link to a given file (or directory)

    ln [source_file] [symbolic_link]

### Softlink

Create a symbolic link to a given file (or directory)

    ln -s [source_file] [symbolic_link]

To delete/remove symbolic links use either the unlink or rm command.

    unlink symlink_to_remove

Using the `rm` command achieves the same

    rm symlink_to_remove

---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
