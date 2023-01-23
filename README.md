# Importing data from multiple files - datastore vs for loop
[![Open in MATLAB Online](https://www.mathworks.com/images/responsive/global/open-in-matlab-online.svg)](https://matlab.mathworks.com/open/github/v1?repo=toshiakit/datastore_vs_for_loop)

_Created with R2022b. Compatible with R2021b and later releases_

It is very common to use for loop when people try to import data from multiple files, but [datastore](https://www.mathworks.com/help/matlab/datastore.html) is as much as 2x faster than using a loop in an example I would like to share. 

* Using a loop: 4.09 sec
* Using datastore: 1.80 sec

To do this comparison, I used Popular Baby Names dataset. I downloaded it and unzipped into a folder named "names". Inside this folder are text files named 'yob1880.txt' through 'yob2021.txt'.

| filenames   | 
| ----------- | 
| yob1880.txt | 
| yob1881.txt |
| yob1882.txt |
| yob1883.txt |
| ...         |
| yob2021.txt |

Let's set up common variables.
```
loc = "names/*.txt";
vars = ["name","sex","births"];
```
## Using for loop
Here is a typical approach using for loop to read individual files. In this case, we also extract year from the file name and add it to the table as a column.
```
tic;
s = dir(loc);                                           % get content of current folder
filenames = arrayfun(@(x) string(x.name), s);           % extract filenames
names = cell(numel(filenames),1);                       % pre-allocate a cell array
for ii = 1:numel(filenames)                             
    filepath = "names/" + filenames(ii);                % define file path
    tbl = readtable(filepath,TextType="string");        % read data from tile
    tbl.Properties.VariableNames = vars;                % add variable names
    year = erase(filenames(ii),["yob",".txt"]);         % extrat year from filename
    tbl.year = repmat(str2double(year),height(tbl),1);  % add column 'year' to the table
    names{ii} = tbl;                                    % add table to cell array
end
names = vertcat(names{:});                              % extract and concatenate tables
toc
```
![Elapsed time - for loop](https://github.com/toshiakit/datastore_vs_for_loop/blob/main/time_loop.png)
```
head(names)
```
![Table](https://github.com/toshiakit/datastore_vs_for_loop/blob/main/table.png)
## Using datastore 
This time, we would do the same thing using datastore and use transform to add the year as a column.
```
tic
ds = datastore(loc,VariableNames=vars,TextType="string"); % set up datastore
ds = transform(ds, @addYearToData, 'IncludeInfo', true);  % add year to the data
names = readall(ds);                                      % extract data into table
toc
```
![Elapsed time - datastore](https://github.com/toshiakit/datastore_vs_for_loop/blob/main/time_datastore.png)
```
head(names)
```
![Table](https://github.com/toshiakit/datastore_vs_for_loop/blob/main/table.png)

Helper function to extract years from the filenames
```
function [data, info] = addYearToData(data, info)
    [~, filename, ~] = fileparts(info.Filename);
    data.year(:, 1) = str2double(erase(filename,"yob"));
end
```

_Copyright 2023 The MathWorks, Inc._
