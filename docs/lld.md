# ENotes: Low-Level Design

For the big picture see the [High-Level Design](hld.md).

## 1. Database

```sql
CREATE TABLE uploads (
  id INT AUTO_INCREMENT PRIMARY KEY,
  file_name VARCHAR(255) NOT NULL,
  file_type VARCHAR(50) NOT NULL,
  file_size INT NOT NULL,
  upload_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

The table is a log of uploads; the list on the home page is read from the `uploads/` folder with `glob("uploads/*")`.

## 2. Upload sequence

```mermaid
sequenceDiagram
  actor S as Student
  participant I as index.php
  participant U as upload.php
  participant FS as uploads/
  participant DB as MySQL
  S->>I: choose file, Upload
  I->>U: POST multipart (upload, file)
  U->>U: basename(name), create uploads/ if missing
  U->>U: size ≤ 2 MB? type in jpeg / png / pdf?
  alt valid
    U->>FS: move_uploaded_file
    U->>DB: INSERT INTO uploads (file_name, file_type, file_size)
  end
  U-->>I: redirect index.php?message=…
  I-->>S: show message (escaped with htmlspecialchars)
```

## 3. Download and delete

| Page | Input | Behavior |
|---|---|---|
| `download.php` | `GET file` | `basename()`, check it exists, send `Content-Disposition: attachment` headers, `readfile()` |
| `delete.php` | `POST file_name` | `basename()`, `unlink()` if it exists, redirect with a message |

## 4. Security notes

| Topic | Current state | Improvement |
|---|---|---|
| Path traversal | Prevented with `basename()` | — |
| SQL injection | Prevented with prepared statements | — |
| File type | Uses the browser-reported MIME type | Check the real type with `finfo` |
| Access control | Anyone can delete any file | Add login and ownership |
| Same-name uploads | Overwrite the existing file | Add a unique prefix |
