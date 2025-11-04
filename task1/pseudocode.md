# Pseudocode - Manage Time Use Case

## Overview

pseudocode for the Vacation Tracking System (VTS) "Manage Time" use case, covering both employee and manager workflows.

---

## 1. Submit Vacation Request (Employee Flow)
```text
FUNCTION submitVacationRequest(employeeId)
    // Step 1: Authenticate
    credentials = getEmployeeCredentials()
    IF NOT validateCredentials(credentials) THEN
        displayError("Invalid credentials")
        RETURN
    END IF
    
    // Step 2: Display balance and history
    balance = getVacationBalance(employeeId)
    history = getVacationHistory(employeeId)
    displayBalanceAndHistory(balance, history)
    
    // Step 3: Select category
    category = selectCategory()
    categoryBalance = checkRemainingBalance(employeeId, category)
    
    IF categoryBalance <= 0 THEN
        displayError("No balance available for this category")
        // Loop back to select different category
        RETURN
    END IF
    
    // Step 4: Display calendar and form
    displayCalendarAndForm(category)
    
    // Step 5: Get request details
    formData = getFormData() // dates, hours, title, description
    
    // Step 6: Validate request
    IF NOT validateRequestData(formData) THEN
        displayErrorsHighlighted(formData.errors)
        
        // Step 7: Give option to edit or cancel
        userChoice = promptEditOrCancel()
        
        IF userChoice == "EDIT" THEN
            // Loop back to enter details again
            formData = getFormData()
        ELSE IF userChoice == "CANCEL" THEN
            RETURN
        END IF
    END IF
    
    // Step 8: Save request to database
    request = saveRequestToDatabase(employeeId, category, formData, status="Pending")
    
    // Step 9: Send email notification if approval required
    IF requiresApproval(category) THEN
        managerId = getManagerId(employeeId)
        sendEmailNotification(managerId, request)
    END IF
    
    // Step 10: Return to home page
    displaySuccessMessage("Request submitted successfully")
    redirectToHomePage()
    
END FUNCTION
```

---

## 2. Process Vacation Request (Manager Flow)
```text
FUNCTION processVacationRequest(managerId)
    // Step 1: Authenticate
    credentials = getManagerCredentials()
    IF NOT validateCredentials(credentials) THEN
        displayError("Invalid credentials")
        RETURN
    END IF
    
    // Step 2: Display manager's data and pending requests
    balance = getVacationBalance(managerId)
    history = getVacationHistory(managerId)
    pendingRequests = getPendingRequestsForSubordinates(managerId)
    displayHomePageWithPendingRequests(balance, history, pendingRequests)
    
    // Step 3: Process requests loop
    WHILE hasMoreRequestsToProcess() DO
        
        // Step 4: Select a request to review
        selectedRequest = selectRequest(pendingRequests)
        
        // Step 5: Display request details
        requestDetails = getRequestDetails(selectedRequest.id)
        displayRequestDetails(requestDetails)
        
        // Step 6: Manager makes decision
        decision = getManagerDecision() // "APPROVE" or "DENY"
        
        IF decision == "APPROVE" THEN
            // Step 7a: Approve path
            updateRequestStatus(selectedRequest.id, status="Approved")
            sendEmailNotification(requestDetails.employeeId, "approved", reason=null)
            
        ELSE IF decision == "DENY" THEN
            // Step 7b: Deny path
            rejectionReason = getManagerRejectionReason()
            updateRequestStatus(selectedRequest.id, status="Rejected", reason=rejectionReason)
            sendEmailNotification(requestDetails.employeeId, "rejected", reason=rejectionReason)
        END IF
        
        // Step 8: Return to home page
        displaySuccessMessage("Request processed successfully")
        redirectToHomePage()
        
        // Step 9: Check if manager wants to process more requests
        moreRequests = promptProcessMoreRequests()
        IF NOT moreRequests THEN
            EXIT WHILE
        END IF
        
    END WHILE
    
    displayMessage("All requests processed")
    
END FUNCTION
```

---
