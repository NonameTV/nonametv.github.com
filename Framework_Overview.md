Framework Overview

Contents

    1 Importers
    2 Database
    3 Exporters
    4 Avoiding unnecessary updates

Importers

Importers are used to import data from different sources into the database. There is one importers for each data provider. All importers inherit from the NonameTV::Importer class and thus provide a consistent interface.

Data is imported in batches. A batch is the smallest unit of data that can be identified by the importer. Usually, a batch corresponds to a file provided by the data provider.

All Importers that import data via HTTP inherit from one of NonameTV::Importer::BaseOne, BaseDaily or BaseWeekly. These baseclasses provide helper-functions that handle data provided in a single file per channel, one file per day and channel or one file per week and channel.

The Importers are run via the command-line tool tools/nonametv-import:

    tools/nonametv-import CanalPlus --verbose 


Database

All data is stored in a MySQL-database in a consistent format. All strings are encoded in utf-8 in the database and all times are stored in UTC-timezone. The channels and tables is updated manually (using the mysql frontend of your choice.) The programs and batches tables are updated by the Importers and cannot be updated manually.
Exporters

The Exporters take the data in the database and export it in another format. There are exporters for xmltv-format, json-format and an rss-format.
Avoiding unnecessary updates

tv_grab_se_swedb utilizes http-caching to avoid downloading data that hasn't changed since the last time tv_grab_se_swedb was run. This saves a lot of bandwidth for the server if utilized correctly.

Additionally, some of the Importers take a lot of time to run and we want to avoid Importing data if nothing has changed. There are a number of mechanisms in place to avoid doing unnecessary work:

All Importers use HTTP-caching (if they fetch data via HTTP) and only processes data if it has actually changed since it was last downloaded. To force an update (for example if the Importer-code has been improved), use the parameter --force-update:

    tools/nonametv-import CanalPlus --force-update 

The Exporter only exports data for batches that have actually changed. Furthermore, after exporting a file, the Exporter checks to see if the exported file is different from the previous file for that period. If the new file is identical to the old file, the old file is left untouched.

After the export-step, rsync is used to copy the data from the xmltv_staging directory to the xmltv-directory. Once more, only files that have actually changed (detected by checking file contents) are copied. This can be useful during development, since you can do an export, see that something went wrong (for example by using tools/nonametv-xmltv-compare-run and look at the changes). Then revert your changes and run export again. Since the newly exported files are then identical to the files in the xmltv/directory, the next run of rsync will not copy any files even though the date on the files has been updated. 
