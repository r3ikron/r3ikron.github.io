---
layout: post
title:  "Command-query separation (CQS) Overview"
---

### **Command**  
1. **Validate parameters**: Check the correctness and structure of input values.
2. **Retrieve data from the database**: Fetch necessary data to perform the action.
3. **Execute the required action**: Carry out the intended operation or modification.
4. **Log the activity**: Record the operation details in a log file or event log.
5. **Refresh the view**: Update the user interface or output to reflect the changes.

---

### **Query**  
1. **Validate parameters**: Verify the input for correctness and completeness.
2. **Fetch data from the database**: Retrieve the requested information.
3. **Update the view**: Display the retrieved data or update the UI.

---

### **Validation**  
1. **Verify input values**: Ensure that the provided inputs meet the required criteria.
2. **Filter input**: Sanitize or filter inputs to remove invalid or harmful data.
3. **Ensure compliance with service and persistence constraints**: Confirm that inputs align with business rules and database integrity requirements.

---

### **Notes**
1. **Error Prevention**: By validating parameters up front, potential issues are identified early, avoiding unnecessary operations.
2. **Separation of Concerns**: Validation focuses solely on verifying input correctness before proceeding with the main workflow.
3. **Consistency**: Both commands and queries start with the validation phase to ensure standardized input handling.
