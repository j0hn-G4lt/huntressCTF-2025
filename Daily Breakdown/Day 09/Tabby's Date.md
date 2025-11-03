- Download the stuff
- File enumeration reveals these are Notepad Tabstate files

```r
$ find C -type f | head -20

C/Windows/win.ini
C/Windows/System32/drivers/etc/hosts
C/Windows/system.ini
C/Users/Tabby/Downloads/desktop.ini
C/Users/Tabby/Pictures/desktop.ini
C/Users/Tabby/Documents/desktop.ini
C/Users/Tabby/NTUSER.DAT
C/Users/Tabby/Music/desktop.ini
C/Users/Tabby/AppData/Local/Microsoft/Windows/UsrClass.dat
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/b5154796-9d23-43ce-8a6c-c373e63f22c0.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/04165ca3-c82b-42ca-ab07-0c774ae66efd.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/bcd5d203-1523-4b86-a572-c1c3afded478.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/17de440f-3f69-4d8a-94fe-f3d4b9cf0c3f.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/4f1c96a1-960c-4cee-9751-fe4b4f59fdd0.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/7ba066a2-e0cb-4c06-9339-316411a3da27.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/056941ef-d51d-4e57-9a55-b59d58bf3fcb.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/7458196e-e979-4d94-982a-246fca3db028.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/9bf7ca49-e491-4691-a21a-f3263bb695a2.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/9e96bd4b-4155-4558-b97a-edcdf01d4584.bin
C/Users/Tabby/AppData/Local/Packages/Microsoft.WindowsNotepad_8wekyb3d8bbwe/LocalState/TabState/68fefe2f-a7a6-4afa-b383-7fdc142aadde.bin
```

- `grep` these Notepad TabState files for flag

```sh
find . -name "*.bin" -exec strings -e l {} \; | grep "flag{"
they told me the password is: flag{165d19b610c02b283fc1a6b4a54c4a58}
```
