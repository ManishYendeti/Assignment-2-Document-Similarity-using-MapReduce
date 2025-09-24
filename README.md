# Assignment 2: Document Similarity using MapReduce  

**Course:** Cloud Computing for Data Analysis (ITCS 6190/8190, Fall 2025)  

**Name:** *Manish Yendeti*  
**Student ID:** *801429980*  

---

## Approach and Implementation  

### Mapper Design  
- **Input (Key, Value):**  
  - Key → Byte offset of the current line in the input file  
  - Value → Entire line of text representing a document  

- **Steps Performed:**  
  - The mapper reads one line at a time.  
  - Separates the **document identifier** (first word) from the **document body** (remaining words).  
  - Cleans the body text by lowercasing and removing punctuation.  
  - Builds a **unique vocabulary set** for that document.  
  - Emits pairs in the form `(DocumentID, Word)` so that word sets can be reconstructed later.  

- **Output (Key, Value):**  
  - Key → Document ID  
  - Value → Token (word)  

---

### Reducer Design  
- **Input (Key, Values):**  
  - Key → Document ID  
  - Value → Collection of tokens from that document  

- **Processing Logic:**  
  - Rebuilds the **word set** for each document.  
  - Produces all possible **unique document pairs**.  
  - For every pair `(DocX, DocY)`, calculates:  
    - Intersection: count of common words  
    - Union: count of all distinct words  
    - Jaccard Similarity = `|Intersection| / |Union|`  

- **Output Example:**  
  ```
  DocA, DocB Similarity: 0.40
  ```

---

### End-to-End Flow  
1. **Input Loading** → Raw documents placed in HDFS.  
2. **Map Phase** → Converts documents into sets of tokens.  
3. **Shuffle & Sort** → Groups all tokens under each document ID.  
4. **Reduce Phase** → Creates document pairs and calculates similarity.  
5. **Final Output** → Jaccard similarity results written back to HDFS.  

---

## Setup and Execution  

**Move to the Assignment Directory:**  
```bash
C:\ManishFall 2025\cloud computingassignment - 2
```

#### 1. Start Hadoop Environment  
```bash
docker compose up -d
```

#### 2. Build Project  
```bash
cd "C:\ManishFall 2025\cloud computingassignment - 2"
mvn clean package
```

#### 3. Transfer JAR to ResourceManager Container  
```bash
docker cp target/DocumentSimilarity-1.0-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

#### 4. Copy Input Dataset into Container  
```bash
docker cp datasets/small_dataset.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

#### 5. Access the Container  
```bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

#### 6. Create Input Directory in HDFS  
```bash
hadoop fs -mkdir -p /input/data
hadoop fs -put ./small_dataset.txt /input/data
```

#### 7. Run the MapReduce Program  
```bash
hadoop jar DocumentSimilarity-1.0-SNAPSHOT.jar com.example.controller.Controller /input/data/small_dataset.txt /output1
```

#### 8. Display Output in HDFS  
```bash
hadoop fs -cat /output1/*
```

#### 9. Bring Output to Local System  
```bash
hdfs dfs -get /output1 /opt/hadoop-3.2.1/share/hadoop/mapreduce/
exit
```

#### From host machine:  
```bash
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output1/ "C:\ManishFall 2025\cloud computingassignment - 2\output"
```

---

## Challenges and Fixes  

### 1. Removing Duplicate Pairs  
**Issue:** Both (DocA, DocB) and (DocB, DocA) were appearing.  
**Fix:** Sorted document IDs before pairing to ensure each pair is only output once.  

---

### 2. Dealing with Case & Punctuation  
**Issue:** Words like *Cloud*, *cloud,* and *cloud.* were all counted separately.  
**Fix:** Normalized by lowercasing and stripping punctuation so they are treated as one token.  

---

### 3. Runtime Differences with Node Count  
**Issue:** Running on a single node was slower compared to multiple nodes, which initially looked like an error.  
**Fix:** Understood this slowdown was expected due to reduced parallelism and documented it in performance notes.  

---

## Input / Output  

#### Input Example:  
```bash
DocA This document talks about cloud computing and big data
DocB Big data analysis is part of cloud computing assignment
DocC Hadoop and MapReduce are used for large scale data processing
```

#### Sample Output:  
```bash
DocA, DocB Similarity: 0.40
DocA, DocC Similarity: 0.25
DocB, DocC Similarity: 0.30
```
