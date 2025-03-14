
# Power Washing Your Data: Using SAS Data Quality Steps in SAS Studio Flows

* [Exercise Description](#exercise-description)
* [Log in to SAS Viya](#log-in-to-sas-viya)
* [Create a SAS Studio Flow](#create-a-sas-studio-flow)
    * [Remove Duplicates with Survivorship Snippet](#remove-duplicates-with-survivorship-snippet)
* [TBD](#tbd)

## Exercise Description

Update your data cleaning process with point-and-click data quality steps in SAS Studio Flows! In this hands-on workshop, you'll learn how to build a flow that deduplicates, standardizes, parses, and combines your data with SAS Data Quality steps and code snippets.

## Log in to SAS Viya

Open a new window in the *Google Chrome* browser and select the **SAS Viya** bookmark.

* ID: **student**
* Password: **Metadata0**

Select **No** when prompted about accepting *Admin* privileges.

## Create a SAS Studio Flow

1. Select ![Viya Menu Selector](img/HamburgerMenu.png) **&#10132; Develop Code and Flows** to open *SAS Studio*.
1. Select **New &#10132; Flow**.
1. Select ![Steps Menu Icon](img/StepsIcon.png) to view the **Steps** pane.

   ![New Flow](img/NewFlow.png)

1. Expand the *Data (Input and Output)* section of the *Steps* pane. Double-click the **Table** step to add it to the flow canvas.
1. Select the **Table**  node on the flow canvas. In the **Table Properties** section, select the following:

   * Library: **DQWS**
   * Table name: **CUSTOMERS**

   ![Customers Table](img/CustomersTable.png)

1. Click **Preview Data** to view a subset of rows from the table. If needed, click ![Maximize](img/Maximize.png) to maximize the preview.

   ![Customers Preview](img/CustomersPreview.png)

   CUSTOMERS has **60** rows and **5** columns. Note that each customer appears three times with different name formats and varying additional column information. We would like to deduplicate this data by combining unique values into one row per customer.

### Remove Duplicates with Survivorship Snippet

1. Expand the *Develop* section of the *Steps* pane. Double-click the **SAS Program** step to add it to the flow canvas.
1. Drag the *SAS Program* step to the right of *CUSTOMERS*. Use your mouse to draw an arrow connecting *Customers* to the *SAS Program* step.

   ![Add Program Step](img/AddProgramStep.png)

1. Select ![Snippets Icon](img/SnippetsIcon.png) to open the **Snippets** pane.
1. Expand **Standard &#10132; Data Quality** and double-click **Survivorship** to open the *Survivorship* snippet in a new tab.

   ![Survivorship Snippet](img/SurvivorshipSnippet.png)

   Per the snippet documentation: *The snippet shows examples of how to use the %dqsurvr autocall macro to identify a surviving record from a group of records in a cluster. It also shows different methods to compose and indicate the surviving record.*

1. Copy the first example that calls the %dqsurvr macro **(lines 52 - 59)** and paste it into the SAS Program node's *Code* tab.

   <p><details markdown="block">

   <summary>Click to view or copy the required section of code.</summary>

   ```bash
   %dqsurvr (inTable=contacts,
              outTable=work.contacts_out,
              clusterColumn=cluster_id,
              rowRule1=(max,ID),
              firstColumnRule1=(highocc,State),
              firstColumnRuleAppliedCols=(Zipcode),
              secondColumnRule1=(not_missing, Address),
              keepDuplicates=0);
    ```

   </details></p>

   ![Survivorship Example](img/SurvivorshipExample.png)

1. Edit the macro parameters as follows:

   * inTable=**&_input1**
   * outTable=**&_output1**
   * clusterColumn=**CustomerID**
   * rowRule1=**(longest, Name)**
     >The surviving record will be chosen by the longest **Name** value, because this should be the fullest version of the customer's name.
   * firstColumnRule1=**(not_missing, Birthday)**
     > The surviving record should store the first non-missing birthday value from the cluster records.
   * **Delete firstColumnRuleAppliedCols=(Zipcode)**
   * secondColumnRule1=**(not_missing, Phone)**
     > The surviving record should store the first non-missing Phone value from the cluster records.
   * **Add a comma after** keepDuplicates=**0**
   * **Add generateDistinctSurvivor=1** after keepDuplicates=**0**

   <p><details markdown="block">

   <summary>Click to view or copy the edited macro call.</summary>

   ```bash
   %dqsurvr (inTable=&_input1,
              outTable=&_output1,
              clusterColumn=CustomerID,
              rowRule1=(longest, Name),
              firstColumnRule1=(not_missing, Birthday),
              secondColumnRule1=(not_missing, Phone),
              keepDuplicates=0,
              generateDistinctSurvivor=1);
    ```

   </details></p>

   ![Survivorship Edited](img/SurvivorshipEdited.png)

1. On the **Node** tab, re-name the node to **Surviving Records**.

   ![Rename Program Node](img/RenameProgramNode.png)

1. Select ![Save Button](img/SaveButton.png) to save the flow.
1. Navigate to **SAS Content &#10132; Public**. Save the file as **DQ_HOW.flw**.
1. Select ![Run](img/Run.png) on the flow toolbar to run the flow.
1. After the low has run successfully, click the shaded **output port** on *Surviving Records*.
1. Click **Preview Data** to review the output.

   ![Survivorship Output](img/SurvivorshipOutput.png)

   The output table has **20** rows and **5** columns. Duplicate records were accurately condensed to store all customer data in one record.

## TBD