# [Path Traversal leading to Arbitrary File Deletion] in [Student Result Management System] <= [v1.0]

**BUG Author:** Kazi Hashibur Rahman  
**Product:** Student Result Management System (SRMS)  
**Vendor:** https://www.sourcecodester.com/  
**Software Link:** https://www.sourcecodester.com/srms-makumbusho  

---

**Vulnerability Details**  
**Type**: Path Traversal leading to Arbitrary File Deletion  
**Vulnerability Type:** Path Traversal leading to Arbitrary File Deletion (CWE-22, CWE-73)  
**Severity:** HIGH (CVSS: 8.6)  

**Affected Components / Files**:
- `admin/core/drop_student.php`
- `academic/core/drop_student.php`
- `admin/core/update_student.php`
- `academic/core/update_student.php`

---

**Vulnerable Parameter**:
- `img` (GET parameter)
- `old_photo` (POST parameter)

**Vulnerable URL Example**:
- `http://localhost/srms/script/admin/core/drop_student?id=RG001&img=avator_1710936891.jpg`

**Root Cause**

The application directly concatenates user-controlled input into filesystem operations without validation or sanitization:

```php
unlink('images/students/' . $_GET['img']);
```
**`admin/core/update_student.php` Line 52**
<img width="985" height="275" alt="image" src="https://github.com/user-attachments/assets/e406b467-b5da-4f1f-bc47-067fb999b3c8" />
There is no restriction, whitelist, or normalization applied to the filename, allowing attackers to manipulate file paths using directory traversal sequences.

**Impact**
- Delete arbitrary files on the server
- Perform directory traversal outside intended upload folder
- Delete sensitive application files (e.g., config.php or .htaccess,)
- Disrupt database connectivity (if config files are removed)

**Description:** The Student Results Management System (SRMS) has a path traversal vulnerability in its file deletion functionality. The application accepts user-controlled input via the `img` and `old_photo` parameters and uses this input directly in filesystem operations without any validation or modification.
An attacker can modify these parameters to include directory traversal sequences such as (../), which allows them to access any location on the server filesystem beyond the target directory (images/students/).
As a result, an attacker can delete any file outside the target directory, including critical application files such as configuration files.

## Proof of Concept (PoC)

1. **Create a test file in the server directory: `test-delete.txt`**
2. **Send the following request or trying to delete any student:**  
- `GET /srms/script/admin/core/drop_student?id=RG001&img=../../test-delete.txt HTTP/1.1`  
- `Host: localhost`
<img width="974" height="482" alt="image" src="https://github.com/user-attachments/assets/b105a079-79c9-43f7-87ea-a19f9653f153" />

3. **The application processes the `img` parameter without validation and executes:**  
- `unlink('images/students/../../test-delete.txt');`
4. **Result:**
- The file `test-delete.txt` is successfully deleted.
- Further testing confirms traversal is possible (e.g., `../config.php` can also be deleted).

## Suggested Remediation
- Implement strict input validation for file parameters
- Use `basename()` to sanitize filenames
- Restrict file operations to allowed directory only
- Implement whitelist of valid filenames from database
- Reject input containing traversal patterns (`../`)

## Additional Information
**Refer to:**
- OWASP Path Traversal Guide
- CWE-22: Path Traversal
- CWE-73: External Control of File Name or Path

This vulnerability requires immediate attention due to its potential to cause significant data loss and application downtime.
