### **Solution**

#### **Overview**
1. **Input File**: The file `transaction-log.txt` contains JSON objects, each representing a transaction.
2. **Objective**:
   - Filter transactions where the `symbol` is `TSLA` and the `side` is `sell`.
   - Extract the `order_id` for these transactions.
   - Submit an HTTP GET request to `https://example.com/api/:order_id` for each filtered `order_id`.
   - Write the responses from the HTTP requests to `./output.txt`.

3. **Constraints**:
   - The solution must be implemented in a single CLI command.
   - The machine is running Ubuntu 24.04, so tools like `jq`, `curl`, and shell utilities are available.

4. **Notes**:
   If `jq` or `curl` are not installed, they can be installed using:
   ```bash
   sudo apt update && sudo apt install jq curl -y
   ```
   Alternatively, tools like `awk` (for JSON parsing) or `wget` (for HTTP requests) can be used, though they may require additional scripting.

---

#### **Construct the Command**
The following command achieves the objective:

```bash
jq -r 'select(.symbol == "TSLA" and .side == "sell") | .order_id' ./transaction-log.txt | \
xargs -I {} curl -s "https://example.com/api/{}" >> ./output.txt
```

---

#### **Explanation of the Command**
1. **`jq` Command**:
   - `jq` is used to parse and filter the JSON data in `transaction-log.txt`.
   - The expression `select(.symbol == "TSLA" and .side == "sell")` filters transactions where the `symbol` is `TSLA` and the `side` is `sell`.
   - The `.order_id` extracts the `order_id` field from the filtered transactions.
   - The `-r` flag ensures that the output is raw text (not JSON-encoded).

2. **`xargs` Command**:
   - `xargs` takes the output of `jq` (the list of `order_id`s) and passes each one as an argument to the `curl` command.
   - The `-I {}` placeholder is replaced with each `order_id` value.

3. **`curl` Command**:
   - `curl` sends an HTTP GET request to the specified URL (`https://example.com/api/:order_id`).
   - The `-s` flag suppresses progress/output to keep the command clean.

4. **Output Redirection**:
   - The `>> ./output.txt` appends the response from each HTTP request to the file `./output.txt`.

---

#### **Example Execution**
Given the input file `transaction-log.txt`, the command will:
1. Identify the following `order_id`s:
   - `12346` (from the first `TSLA` sell transaction)
   - `12362` (from the second `TSLA` sell transaction)
2. Submit HTTP GET requests to:
   - `https://example.com/api/12346`
   - `https://example.com/api/12362`
3. Append the responses from these requests to `./output.txt`.

---

#### **Notes and Assumptions**
1. **Error Handling**:
   - If any HTTP request fails, it will not stop the execution of subsequent requests.
   - The `curl` command uses the `-s` flag to suppress errors, but you can add error handling if needed (e.g., using `--fail`).

2. **File Overwriting**:
   - The `>>` operator appends to `./output.txt`. If the file already exists, its contents will remain intact, and new data will be added at the end.

3. **Dependencies**:
   - Ensure that `jq` and `curl` are installed on the system. These tools are typically pre-installed on Ubuntu systems.