# **Generating EJScreen TSDF Facility Proximity**

EJScreen uses Apache Hadoop pig scripts to generate Treatment, Storage, and Disposal Facilities (TSDF) facility proximity. The Pig scripts were developed using Esri's [GIS Toolkit for Hadoop](https://esri.github.io/gis-tools-for-hadoop/) toolkit. It was run in an AWS EMR cluster environment. The source data came from EPA's Office of Enforcement and Compliance Assurance (OECA). The proximity process involves Pre-Hadoop processing, running Hadoop Pig scripts, and Post-Hadoop processing. The end results are Census block-group based proximity scores.

**Source:**

The source dataset, tsdf\_br2021\_feb2023.xlsx, contains a list of Resource Conservation and Resource Conservation and Recovery Act (RCRA) handlers that are either operating TSDFs or reporting Large Quantity Generators (LQGs) in the 2021 Biennial Report (BR). Provided in February 2023 by Office of Enforcement and Compliance (OECA).

**Pre-Hadoop Processing:**

- Import tsdf\_br2021\_feb2023.xlsx to table in geodatabase called TSDF\_work.gdb -- TSDF\_BR2021\_0323\_all
- Determine number TSDF and BR records. 666 "Operating TSDF" records, out of 17,025 LQG BR reporters.
- Also check for duplicate facilities. No duplicates were found.
- Geocode TSDF\_BR2021\_0223\_all to create TSDF\_BR2021\_0223\_forEJ. Check for points outside US and PR. Drop records outside US states and PR leaving total of 17,013. Dropped 12 from GU, MP, VI.
- Create output table (TSDF\_BR2019\_0223\_forHadoop.csv). The export table structure is (EPA\_ID, LATITUDE, LONGITUDE, CWEIGHT); note that CWEIGHT = 1 for all records.

**AWS Hadoop Processing:**

- Start an AWS EMR cluster.
- Run Step 1 for each of 32 state groups to generate weighted distances for facility-block pairs. See **TSDF\_US01\_proximity\_Step\_1.txt** for Pig script example.
- Run Step 2 for each of 32 state groups to generate block group summary for each subgroup. See **TSDF\_US01\_proximity\_Step\_2.txt** for Pig script example.
- Use Athena to create BG-level results tables and export BG\_Scores\_01.csv, etc. to OutputfromHadoop folder.
- Repeat all steps for each subgroup.

**Post-Hadoop Processing:**

- Combine all text files in OutputfromHadoop folder. Copy \*\_bgscores.csv: tsdf\_bgscores\_us.csv
- Prep with text editor (Capitalized first header row and remove all other header rows).
- Convert tsdf\_bgscores.csv to xlsx; make sure BLKGRP is text.
- Port tsdf\_bgscores.xlsx file into geodatabase table (TSDF\_Work.gdb/TSDF\_BG\_Scores\_US)
- Rename columns to STCNTRBG and BG\_SCORE.
- Provide datasets for testing--Create TSDFProximity\_Testing.gdb
- Include US\_BG\_Scores\_Final, and TSDF\_BR2021\_0223\_forEJ
- Add US\_TSDFProx\_BG with BG shapes and BG\_SCORE, and set Null Scores to 0.

**EPA Disclaimer**

The United States Environmental Protection Agency (EPA) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. EPA has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by EPA. The EPA seal and logo shall not be used in any manner to imply endorsement of any commercial product or activity by EPA or the United States Government.
