# Weekly progress report
* [Report](./Weekly%20progress%20report/report.md)



# RefCommitDiff
Language-agnostic Refactoring Detection

## commit_diff_db
Testing Phase Version.

Considering the data volume in the later stages, I chose MySQL instead of SQLite.

Currently, I am using `git log --all`. In the future, I may consider adding options to exclude merge commits and other entries.

### Tables in commit_diff_db

There are six tables `repository`, `refactor_keywords`, `mbassador_original_commits`,`mbassador_finergit_original_mapping`,`mbassador_finergit_commits_hayashi`, and `mbassador_diff_lines` in commit_diff_db.
```shell-session
mysql> show tables;
+-------------------------------------+
| Tables_in_commit_diff_db            |
+-------------------------------------+
| mbassador_diff_lines                |
| mbassador_finergit_commits_hayashi  |
| mbassador_finergit_original_mapping |
| mbassador_original_commits          |
| refactor_keywords                   |
| repository                          |
+-------------------------------------+
```

### Table `mbassador_original_commits`

The schema of table `mbassador_original_commits` is as follows.

```shell-session
mysql> desc mbassador_original_commits;
+------------------------+--------------+------+-----+---------+-------+
| Field                  | Type         | Null | Key | Default | Extra |
+------------------------+--------------+------+-----+---------+-------+
| commit_id              | varchar(40)  | NO   | PRI | NULL    |       |
| commit_id_short        | varchar(7)   | YES  |     | NULL    |       |
| repository_id          | int          | YES  |     | NULL    |       |
| repository_name        | varchar(255) | YES  |     | NULL    |       |
| commit_message_subject | text         | YES  |     | NULL    |       |
| is_file_modified       | tinyint(1)   | NO   |     | NULL    |       |
| is_code_file_modified  | tinyint(1)   | NO   |     | NULL    |       |
| commit_message_body    | text         | YES  |     | NULL    |       |
| refactor_keyword_id    | int          | YES  |     | NULL    |       |
| commit_date            | timestamp    | NO   |     | NULL    |       |
+------------------------+--------------+------+-----+---------+-------+
```

- `commit_id` Represents the original repository commit ID (SHA1 hash value).
- `commit_id_short`, will be deleted later.
- `repository_id`, Repository ID, linked to the repository table.
- `repository_name`, will be deleted later.
- `commit_message_subject`, The subject or title of the commit message.
- `is_file_modified`, Indicates whether a file was modified.
- `is_code_file_modified`, Indicates whether a code file was modified.
- `commit_message_body`, Detailed commit message describing the changes in the commit.
- `refactor_keyword_id`, Refactoring keyword ID, linked to the refactor_keywords table.
- `commit_date`, UTC time without timezone information.

**`is_code_file_modified`** can help filter out commits that do not contain code file (e.g..java) changes.

### Table `mbassador_finergit_original_mapping`

The schema of table `mbassador_finergit_original_mapping` is as follows.

```shell-session
mysql> desc mbassador_finergit_original_mapping;
+--------------------+-------------+------+-----+---------+-------+
| Field              | Type        | Null | Key | Default | Extra |
+--------------------+-------------+------+-----+---------+-------+
| commit_id          | varchar(40) | NO   | PRI | NULL    |       |
| original_commit_id | varchar(7)  | NO   |     | NULL    |       |
| commit_date        | timestamp   | YES  |     | NULL    |       |
+--------------------+-------------+------+-----+---------+-------+
```

- `commit_id` Represents the commit ID of a repository tokenized using FinerGit.
- `original_commit_id`, Represents the original repository commit ID (SHA1 hash value).
- `commit_date`, UTC time without timezone information.
  
This table stores the mapping between the commit IDs of the original repository and the tokenized repository.

### Table `mbassador_diff_lines`


The schema of table `mbassador_diff_lines` is as follows.

```shell-session
mysql> desc mbassador_diff_lines;
+---------------+---------------+------+-----+---------+----------------+
| Field         | Type          | Null | Key | Default | Extra          |
+---------------+---------------+------+-----+---------+----------------+
| id            | int           | NO   | PRI | NULL    | auto_increment |
| commit_id     | varchar(40)   | NO   |     | NULL    |                |
| file_name     | varchar(255)  | NO   |     | NULL    |                |
| file_path     | text          | NO   |     | NULL    |                |
| commit_date   | timestamp     | NO   |     | NULL    |                |
| hunk_id       | int           | NO   |     | NULL    |                |
| hunk_header   | text          | NO   |     | NULL    |                |
| line_id       | int           | NO   |     | NULL    |                |
| change_type   | enum('+','-') | NO   |     | NULL    |                |
| token_type    | varchar(50)   | YES  |     | NULL    |                |
| token_value   | text          | NO   |     | NULL    |                |
| refactor_type | varchar(50)   | YES  |     | NULL    |                |
+---------------+---------------+------+-----+---------+----------------+
```

The example of table `mbassador_diff_lines` is as follows.

```shell-session
mysql> select * from  mbassador_diff_lines limit 5;
+----+------------------------------------------+-------------------------------------------------------+---------------------------------------+---------------------+---------+-----------------+---------+-------------+----------------+-----------------------------------------------------------------------------------------------------------------------+---------------+
| id | commit_id                                | file_name                                             | file_path                             | commit_date         | hunk_id | hunk_header     | line_id | change_type | token_type     | token_value                                                                                                           | refactor_type |
+----+------------------------------------------+-------------------------------------------------------+---------------------------------------+---------------------+---------+-----------------+---------+-------------+----------------+-----------------------------------------------------------------------------------------------------------------------+---------------+
|  1 | 10517dce9b4a86b9f49c24a86bb4b84a76eb545b | AbstractConcurrentSet#private_boolean_insert(T).mjava | src/main/java/net/engio/mbassy/common | 2015-08-31 21:02:31 |       0 | @@ -1,3 +1,6 @@ |       0 | +           |                | /**                                                                                                                   | NULL          |
|  2 | 10517dce9b4a86b9f49c24a86bb4b84a76eb545b | AbstractConcurrentSet#private_boolean_insert(T).mjava | src/main/java/net/engio/mbassy/common | 2015-08-31 21:02:31 |       0 | @@ -1,3 +1,6 @@ |       1 | +           |                |  * Inserts a new element at the head of the set. Note: This method is expected to be synchronized by the calling code | NULL          |
|  3 | 10517dce9b4a86b9f49c24a86bb4b84a76eb545b | AbstractConcurrentSet#private_boolean_insert(T).mjava | src/main/java/net/engio/mbassy/common | 2015-08-31 21:02:31 |       0 | @@ -1,3 +1,6 @@ |       2 | +           | JAVADOCCOMMENT |  */                                                                                                                   | NULL          |
|  4 | 08f6982fa51b4712bea086347e5cfd76ee94bae2 | AbstractConcurrentSet#private_boolean_insert(T).mjava | src/main/java/net/engio/mbassy/common | 2015-02-25 11:32:43 |       0 | @@ -1,5 +1,5 @@ |       1 | -           | VOID           | void                                                                                                                  | NULL          |
|  5 | 08f6982fa51b4712bea086347e5cfd76ee94bae2 | AbstractConcurrentSet#private_boolean_insert(T).mjava | src/main/java/net/engio/mbassy/common | 2015-02-25 11:32:43 |       0 | @@ -1,5 +1,5 @@ |       2 | +           | BOOLEAN        | boolean                                                                                                               | NULL          |
+----+------------------------------------------+-------------------------------------------------------+---------------------------------------+---------------------+---------+-----------------+---------+-------------+----------------+-----------------------------------------------------------------------------------------------------------------------+---------------+
```


- `id`, Auto-increment primary key.
- `commit_id`, Represents the commit ID of a repository tokenized using FinerGit.
- `file_name`, File name of the .mjava file, without path information.
- `file_path`, File path of the .mjava file.
- `commit_date`, UTC time without timezone information.
- `hunk_id`, Unique identifier for a hunk (Hunk ID), starting from 0, indicating the sequence number of each hunk in the diff.
- `hunk_header`, Header information of the hunk.
- `line_id`, Unique identifier for a line (Line ID), starting from 0, representing the relative line number in the diff file.
- `change_type`, Type of change (+ for additions, - for deletions).
- `token_type`, Type of token. If empty, it is a comment and can be ignored. JAVADOC comments are also considered comments and can be ignored.
- `token_value`, Refactoring keyword ID, linked to the refactor_keywords table.
- `refactor_type`, Value of the token.

### Table `mbassador_finergit_commits_hayashi`

Using the same logic as Hayashi-sensei, the specific details of method renaming, method migration, and method parameter changes are inferred by detecting changes in file names after the repository has been tokenized by FinerGit.

The schema of table `mbassador_finergit_commits_hayashi` is as follows.

```shell-session
mysql> desc mbassador_finergit_commits_hayashi;
+-----------------------+--------------+------+-----+---------+----------------+
| Field                 | Type         | Null | Key | Default | Extra          |
+-----------------------+--------------+------+-----+---------+----------------+
| id                    | int          | NO   | PRI | NULL    | auto_increment |
| commit_id             | varchar(40)  | NO   |     | NULL    |                |
| original_commit_id    | varchar(40)  | NO   |     | NULL    |                |
| file_similarity_score | int          | NO   |     | NULL    |                |
| change_type           | varchar(30)  | NO   |     | NULL    |                |
| change_type_info      | text         | NO   |     | NULL    |                |
| old_file_path         | varchar(255) | NO   |     | NULL    |                |
| new_file_path         | varchar(255) | NO   |     | NULL    |                |
+-----------------------+--------------+------+-----+---------+----------------+
```

- `id`, Auto-increment primary key.
- `commit_id` Represents the commit ID of the tokenized repository, mbassador.
- `original_commit_id`, Represents the original repository commit ID.
- `file_similarity_score`, Represents the similarity score assigned by Git when it detects a file rename or movement, indicating the percentage of similarity between the original file and the new file.
- `change_type`, A high-level label that categorizes the type of change detected in the file.
- `change_type_info`, A detailed description of the detected change.
- `old_file_path`, Path to the file in its original state before the commit.
- `new_file_path`, Path to the file after the commit.

### Table `refactor_keywords`

The schema of table `refactor_keywords` is as follows.

```shell-session
mysql> desc refactor_keywords;
+-----------------+-------------+------+-----+---------+----------------+
| Field           | Type        | Null | Key | Default | Extra          |
+-----------------+-------------+------+-----+---------+----------------+
| id              | int         | NO   | PRI | NULL    | auto_increment |
| keyword_type    | varchar(50) | NO   |     | NULL    |                |
| keyword_content | varchar(50) | NO   |     | NULL    |                |
+-----------------+-------------+------+-----+---------+----------------+
```

- `id`, Keyword ID, auto-increment primary key.
- `keyword_type`, Refactoring type keyword (e.g., "rename method," "change parameter type").
- `keyword_content`, Content of the keyword: Keywords or phrases in commit messages that can be associated with this refactoring type.

### Table `repository`

The schema of table `repository` is as follows.

```shell-session
mysql> desc repository;
+-----------------+--------------+------+-----+---------+----------------+
| Field           | Type         | Null | Key | Default | Extra          |
+-----------------+--------------+------+-----+---------+----------------+
| id              | int          | NO   | PRI | NULL    | auto_increment |
| repository_url  | text         | NO   |     | NULL    |                |
| repository_name | varchar(255) | YES  |     | NULL    |                |
| language        | varchar(50)  | NO   |     | NULL    |                |
+-----------------+--------------+------+-----+---------+----------------+
```

- `id`, Repository ID, auto-increment primary key.
- `repository_url`, Repository URL.
- `repository_name`, Name of the repository.
- `language`, Primary programming language of the repository (e.g., Java, Python).


## Patterns of each refactoring type in commits

systematically summarize the behavioral patterns of each refactoring type, progressing step by step from the:

`original commit level → file level → method level → line level (hunk level)`

Based on the summarized patterns, design rules or automated tools to detect refactoring instances.





## RefactoringMiner-supported refactoring types

Below is the classification of RefactoringMiner-supported refactoring types by **subject** (method, variable, class, attribute, etc.), with each category further divided by subject type and related operation types.

---

### **Method-Related Refactorings**
#### **Addition, Deletion, Renaming, and Moving of Methods**
- **Addition**:
  - Add Method Annotation
  - Add Method Modifier (final, static, abstract, synchronized)
  - Add Thrown Exception Type
- **Deletion**:
  - Remove Method Annotation
  - Remove Method Modifier (final, static, abstract, synchronized)
  - Remove Thrown Exception Type
- **Renaming**:
  - Rename Method
- **Moving**:
  - Move Method
  - Move and Rename Method
  - Move and Inline Method

#### **Adjustments to Method Content**
- **Extraction**:
  - Extract Method
  - Extract and Move Method
- **Inlining**:
  - Inline Method
- **Parameter Adjustments**:
  - Add Parameter
  - Remove Parameter
  - Reorder Parameter
  - Localize Parameter
- **Modification**:
  - Modify Method Annotation
  - Change Method Access Modifier
  - Change Return Type
  - Change Thrown Exception Type
- **Merging and Splitting**:
  - Merge Method
  - Split Method

#### **Specific Refactorings**
- Replace Loop with Pipeline
- Replace Pipeline with Loop
- Replace Conditional With Ternary
- Replace Anonymous with Lambda
- Replace Anonymous with Class

---

### **Variable-Related Refactorings**
#### **Addition, Deletion, Renaming, and Moving of Variables**
- **Addition**:
  - Add Variable Annotation
  - Add Variable Modifier (final)
- **Deletion**:
  - Remove Variable Annotation
  - Remove Variable Modifier (final)
- **Renaming**:
  - Rename Variable
  - Parameterize Variable
- **Moving**:
  - Replace Variable with Attribute

#### **Adjustments to Variable Content**
- **Extraction**:
  - Extract Variable
- **Inlining**:
  - Inline Variable
- **Type Modification**:
  - Change Variable Type
  - Change Parameter Type
- **Merging and Splitting**:
  - Merge Variable
  - Split Variable

---

### **Attribute-Related Refactorings**
#### **Addition, Deletion, Renaming, and Moving of Attributes**
- **Addition**:
  - Add Attribute Annotation
  - Add Attribute Modifier (final, static, transient, volatile)
- **Deletion**:
  - Remove Attribute Annotation
  - Remove Attribute Modifier (final, static, transient, volatile)
- **Renaming**:
  - Rename Attribute
  - Move and Rename Attribute
- **Moving**:
  - Move Attribute
  - Replace Attribute with Variable

#### **Adjustments to Attribute Content**
- **Extraction**:
  - Extract Attribute
- **Inlining**:
  - Inline Attribute
- **Type Modification**:
  - Change Attribute Type
- **Encapsulation**:
  - Encapsulate Attribute
  - Parameterize Attribute
- **Merging and Splitting**:
  - Merge Attribute
  - Split Attribute

---

### **Class-Related Refactorings**
#### **Addition, Deletion, Renaming, and Moving of Classes**
- **Addition**:
  - Add Class Annotation
  - Add Class Modifier (final, static, abstract)
- **Deletion**:
  - Remove Class Annotation
  - Remove Class Modifier (final, static, abstract)
- **Renaming**:
  - Rename Class
  - Move and Rename Class
- **Moving**:
  - Move Class
  - Move Package
  - Split Package
  - Merge Package

#### **Adjustments to Class Structure**
- **Extraction**:
  - Extract Class
  - Extract Subclass
  - Extract Superclass
  - Extract Interface
- **Inlining**:
  - Inline Attribute
- **Hierarchy Adjustments**:
  - Collapse Hierarchy
- **Type Declaration Modification**:
  - Change Type Declaration Kind (class, interface, enum, annotation, record)
- **Merging and Splitting**:
  - Merge Class
  - Split Class

---

### **Package-Related Refactorings**
- Rename Package  
- Move Package  
- Split Package  
- Merge Package  

---

### **Annotation-Related Refactorings**
#### **Method Annotations**
- Add Method Annotation
- Remove Method Annotation
- Modify Method Annotation
#### **Attribute Annotations**
- Add Attribute Annotation
- Remove Attribute Annotation
- Modify Attribute Annotation
#### **Class Annotations**
- Add Class Annotation
- Remove Class Annotation
- Modify Class Annotation
#### **Parameter Annotations**
- Add Parameter Annotation
- Remove Parameter Annotation
- Modify Parameter Annotation
#### **Variable Annotations**
- Add Variable Annotation
- Remove Variable Annotation
- Modify Variable Annotation

---

### **Other Refactorings**
#### **Modifier-Related**
- Add Method Modifier (final, static, abstract, synchronized)
- Remove Method Modifier (final, static, abstract, synchronized)
- Add Attribute Modifier (final, static, transient, volatile)
- Remove Attribute Modifier (final, static, transient, volatile)
- Add Class Modifier (final, static, abstract)
- Remove Class Modifier (final, static, abstract)
- Add Variable Modifier (final)
- Remove Variable Modifier (final)

#### **Test-Related**
- Parameterize Test (JUnit 5 @ParameterizedTest with @ValueSource)
- Assert Throws

#### **Conditions and Exception Handling**
- Split Conditional
- Merge Conditional
- Invert Condition
- Merge Catch
- Add Thrown Exception Type
- Remove Thrown Exception Type

#### **Migration-Related**
- Replace Loop with Pipeline
- Replace Pipeline with Loop
- Replace Conditional With Ternary
- Replace Anonymous with Lambda
- Replace Anonymous with Class
- Replace Generic With Diamond
- Try With Resources

---

