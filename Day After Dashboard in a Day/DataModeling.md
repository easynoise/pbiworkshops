# Data Modeling

 supports the following scenarios by:

1. Promoting a single source of the truth, with greater control over which data is accessed and exposed to creators.

[Learn more about storage modes](https://docs.microsoft.com/power-bi/transform-model/dataflows/dataflows-introduction-self-service)

[Learn more about external tools](https://docs.microsoft.com/power-bi/transform-model/desktop-external-tools)

# Storage modes

## DirectQuery

1. Some of the first things we might notice is that in the bottom right corner of Power BI Desktop is the text **Storage Mode: DirectQuery** and on the left side-rail our **Data** view is now missing.

    ![Missing data view.](./Media/MissingDataView.png)

1. To confirm that our connection to our dataflow is indeed working, we'll now add the following columns below into a **Table** visual to view data on a report page.

    | Table | Column |
    |:----- | :------ |
    | DimCustomer | EmailAddress|
    | DimCustomer | Gender |

    ![Table visual.](./Media/TableVisual.png)

1. While the visual has rendered successfully, we want to confirm that the information is being sent to the source system. To confirm we'll navigate to the **View** tab and then select **Performance analyzer**.

    ![Performance analyzer.](./Media/PerformanceAnalyzer.png)

1. Once the **Performance analyzer** pane is visible, select **Start recording**.
    
    ![Start recording button.](./Media/StartRecording.png)

1. With the **Performance analyzer** now recording, we can hover above the **Table** visual and select the **Analyze this visual** option to refresh our visual. Once complete select the expand/collapse box next to the **Table** visual to expand the results and confirm that a **Direct query** value is present. Select the **Copy query** option and paste the query into a text editor.

    ![Analyze this visual.](./Media/AnalyzeThisVisual.png)

1. From the copied query, we are able to view the **DAX Query** that was sent to the analysis services database instance.

    ```dax
    // DAX Query
    DEFINE
      VAR __DS0Core = 
        SUMMARIZE('DimCustomer', 'DimCustomer'[Gender], 'DimCustomer'[EmailAddress])
    
      VAR __DS0PrimaryWindowed = 
        TOPN(501, __DS0Core, 'DimCustomer'[Gender], 1, 'DimCustomer'[EmailAddress], 1)
    
    EVALUATE
      __DS0PrimaryWindowed
    
    ORDER BY
      'DimCustomer'[Gender], 'DimCustomer'[EmailAddress]
    ```
    [Learn more about DirectQuery guidance](https://docs.microsoft.com/en-us/power-bi/guidance/directquery-model-guidance)

---

### Optional - View event traces

---

One important item of note that was missing from our above query is the [Transact-SQL](https://docs.microsoft.com/learn/modules/introduction-to-transact-sql/) statement for the **Direct query** value. To trace this event we'll use an external tool titled [SQL Server Profiler](https://docs.microsoft.com/sql/tools/sql-server-profiler/sql-server-profiler) to view event traces. We can leverage [external tools in Power BI Desktop](https://docs.microsoft.com/power-bi/transform-model/desktop-external-tools) to view the event traces against our underlying Analysis Services instance.


#### Prerequisite - Register the SQL Server Profiler external tool

1. Download and install [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) or the [Azure Data Studio with the SQL Server Profiler extension](https://docs.microsoft.com/sql/azure-data-studio/extensions/sql-server-profiler-extension?view=sql-server-ver15).

1. Download the registered external tool [SQL Server Profiler External Tools](https://raw.githubusercontent.com/microsoft/pbiworkshops/main/Day%20After%20Dashboard%20in%20a%20Day/Source_Files/SQLProfiler.pbitool.json) (json) file. (Courtesy of Microsoft MVP [Steve Campbell](https://mvp.microsoft.com/PublicProfile/5004099))

1. After the above is downloaded add the local file (SQLProfiler.pbitool.json) to the below file path location. Once complete close and restart your Power BI Desktop application.

    ```
    C:\Program Files (x86)\Common Files\Microsoft Shared\Power BI Desktop\External Tools
    ```

#### DirectQuery event traces

1. From the **Trace Properties** window select the **Events Selection** tab. Within the **Events** view, expand the **Query Processing** group and then select the **DirectQuery End** event. Once complete select the **Run** option to begin tracing events.

    ![Trace properties.](./Media/TraceProperties.png)

1. Return to the Power BI Desktop application and select the **Analyze this visual** option to send another query to the source system.

    ![Analyze this visual refresh.](./Media/AnalyzeThisVisualRefresh.png)

1. Return to the **SQL Server Profiler** application, where you can now see the **DirectQuery end** event including the **Text data** of the SQL query generated by Power BI, the total **Duration** and more.

    ![DirectQuery end.](./Media/DirectQueryEnd.png)

    ```sql
    SELECT
        TOP (501) [t1].[EmailAddress],
        [t1].[Gender]
    FROM
        [DimCustomer] AS [t1]
    GROUP BY
        [t1].[EmailAddress],
        [t1].[Gender]
    ORDER BY
        [t1].[Gender] ASC,
        [t1].[EmailAddress] ASC
    ```

---

## Relationships

1. Create relationships
1. Referential integrity
1. Snowflake schema
1. Star Schema

1. Add a **Table** visualization using the following columns:

| Table | Column |
|:----- | :------ |
| Customer | EmailAddress|
| Internet Sales | Sales Amount |



1. Navigate to the **View** tab and select **Performance analyzer**.

    ![Performance analyzer.](./Media/PerformanceAnalyzer.png)

1. Within the **Performance analyzer** pane, select **Start recording**.
    ![Start recording button.](./Media/StartRecording.png)

1.	Press the Clear option, Refresh visuals and Copy query
1.	Navigate to the Model view, select the Dimension tables, navigate to Advanced and Storage mode to change to Dual