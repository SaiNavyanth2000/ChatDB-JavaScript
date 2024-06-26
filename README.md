# ChatDB-JavaScript

Web version of ChatDB completely run on the browser.
<br>
The main goal of this project is to query any database using natural langauge. <br>
The difference is, it is completely run in the browser and not on any server.<br>
This allows privacy of the DB owners, as no one person/team has access to the DB apart from the DB owner.
<br><br>
Steps to use this notebook on the browser for now:

1. Download the notebook.
2. Open xeus-jupyterlite (https://jupyter-xeus.github.io/xeus-javascript/lab/index.html).
   <br>This is a JavaScript kernel for JupyterLite, powered by Xeus
3. Upload the file to it and run the cells.
4. Modify the first cell to update the query.

# Utilities

## To display results

```javascript
display_data = function (data1, metadata = {}, transient = {}) {
  //TODO: to upgrade to Xeus 4
  ijs.display.display(data1, metadata, transient);
};

update_data = function (data1, metadata = {}, transient = {}) {
  //TODO: to upgrade to Xeus 4
  ijs.display.update_display_data(data1, metadata, transient);
};

display_markdown = function (markdown1, metadata = {}, transient = {}) {
  display_data({ "text/markdown": markdown1 }, metadata, transient);
};
```

## To generate DB Schema String

```javascript
await import("https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js");
console.log("test");
let pyodide = await loadPyodide();
let zipResponse = await fetch(
  "https://github.com/SaiNavyanth2000/public_data/raw/main/hsbc_sample_employee_db.zip"
);
let zipBinary = await zipResponse.arrayBuffer();
pyodide.unpackArchive(zipBinary, "zip");
await pyodide.loadPackage("micropip");
const micropip = pyodide.pyimport("micropip");
await micropip.install("pandas");
await micropip.install("sqlalchemy");
let extract_data = pyodide.runPython(`
from sqlalchemy import text
import sqlalchemy
import pandas as pd
#2.-Turn on database engine
dbEngine=sqlalchemy.create_engine('sqlite:///employees.db', echo=True) 
def extract_data(sql_query):
    with dbEngine.begin() as conn:
        query = text(sql_query)
        df = pd.read_sql_query(query, conn)
        return df.to_json()
extract_data
`);

schema_str = extract_data("pragma table_info(customers)");
schema_str;
```

```diff
{"cid":{"0":0,"1":1,"2":2,"3":3,"4":4,"5":5,"6":6,"7":7,"8":8,"9":9,"10":10,"11":11,"12":12},"name":{"0":"CustomerId","1":"FirstName","2":"LastName","3":"Company","4":"Address","5":"City","6":"State","7":"Country","8":"PostalCode","9":"Phone","10":"Fax","11":"Email","12":"SupportRepId"},"type":{"0":"INTEGER","1":"NVARCHAR(40)","2":"NVARCHAR(20)","3":"NVARCHAR(80)","4":"NVARCHAR(70)","5":"NVARCHAR(40)","6":"NVARCHAR(40)","7":"NVARCHAR(40)","8":"NVARCHAR(10)","9":"NVARCHAR(24)","10":"NVARCHAR(24)","11":"NVARCHAR(60)","12":"INTEGER"},"notnull":{"0":1,"1":1,"2":1,"3":0,"4":0,"5":0,"6":0,"7":0,"8":0,"9":0,"10":0,"11":1,"12":0},"dflt_value":{"0":null,"1":null,"2":null,"3":null,"4":null,"5":null,"6":null,"7":null,"8":null,"9":null,"10":null,"11":null,"12":null},"pk":{"0":1,"1":0,"2":0,"3":0,"4":0,"5":0,"6":0,"7":0,"8":0,"9":0,"10":0,"11":0,"12":0}}
```

# Chat Agent

```javascript
import { OpenAI } from "@langchain/openai";

chat = async function (user_query) {
  const openai_api_base = "https://delta.helloway.workers.dev/v1";
  const openai_api_key = "sk-BGuzv1Xgj1u5WDObw7sYT3BlbkFJrQd1upRgeTbymcina";

  // Initialize a new instance of ChatOpenAI
  const llm = new OpenAI({
    modelName: "gpt-4",
    configuration: {
      baseURL: openai_api_base,
    },
    openAIApiKey: openai_api_key,
    temperature: 0,
  });

  display_markdown("### User Query:");
  const query_str = "### " + user_query;
  display_markdown(query_str);
  display_markdown("*helper steps to answer the question:*");
  prompt_str = `
here is a schema of a 'customers' table saved in an sqlite database. I would like you to understand this table completely,
and understand the following user question about the table and provide any SQL query, that should work for an sqlite database, that would return the 
related records that would answer the user question.
here is the schema:
'{"cid":{"0":0,"1":1,"2":2,"3":3,"4":4,"5":5,"6":6,"7":7,"8":8,"9":9,"10":10,"11":11,"12":12},"name":{"0":"CustomerId","1":"FirstName","2":"LastName","3":"Company","4":"Address","5":"City","6":"State","7":"Country","8":"PostalCode","9":"Phone","10":"Fax","11":"Email","12":"SupportRepId"},"type":{"0":"INTEGER","1":"NVARCHAR(40)","2":"NVARCHAR(20)","3":"NVARCHAR(80)","4":"NVARCHAR(70)","5":"NVARCHAR(40)","6":"NVARCHAR(40)","7":"NVARCHAR(40)","8":"NVARCHAR(10)","9":"NVARCHAR(24)","10":"NVARCHAR(24)","11":"NVARCHAR(60)","12":"INTEGER"},"notnull":{"0":1,"1":1,"2":1,"3":0,"4":0,"5":0,"6":0,"7":0,"8":0,"9":0,"10":0,"11":1,"12":0},"dflt_value":{"0":null,"1":null,"2":null,"3":null,"4":null,"5":null,"6":null,"7":null,"8":null,"9":null,"10":null,"11":null,"12":null},"pk":{"0":1,"1":0,"2":0,"3":0,"4":0,"5":0,"6":0,"7":0,"8":0,"9":0,"10":0,"11":0,"12":0}}'

user query: ${user_query}
please return just the SQL query and nothing else. dont return any extra keywords or explanation.
just return the query
`;

  const res = await llm.call(prompt_str);
  display_markdown("***sql query generated. here is the query:***");
  const text = "**" + res;
  display_markdown(text);
  // download sqlite database
  await import("https://cdn.jsdelivr.net/pyodide/v0.25.0/full/pyodide.js");
  let pyodide = await loadPyodide();
  let zipResponse = await fetch(
    "https://github.com/SaiNavyanth2000/public_data/raw/main/hsbc_sample_employee_db.zip"
  );
  let zipBinary = await zipResponse.arrayBuffer();
  pyodide.unpackArchive(zipBinary, "zip");
  display_markdown("***sqlite database downloaded***");

  //read relevant rows from database
  await pyodide.loadPackage("micropip");
  const micropip = pyodide.pyimport("micropip");
  await micropip.install("pandas");
  await micropip.install("sqlalchemy");
  let extract_data = pyodide.runPython(`
    from sqlalchemy import text
    import sqlalchemy
    import pandas as pd
    #2.-Turn on database engine
    dbEngine=sqlalchemy.create_engine('sqlite:///employees.db') 
    def extract_data(sql_query):
        with dbEngine.begin() as conn:
            query = text(sql_query)
            df = pd.read_sql_query(query, conn)
            print('data fetched')
            return df.to_json()
    extract_data
  `);
  let df_result = extract_data(res);
  display_markdown("***extracted data from the database. here is the data:***");
  console.log(df_result);

  //generate python query

  prompt_str = `
here is a question from a user that you need to answer only based on the given data and nothing else:
user query: ${user_query}

here is an SQL query related to the user question that helps answering the above question. use this query just to help your answer, dont specify about the sql query itself in your answer:
sql_query: ${res}

here is the result from executing the above SQL query:
${df_result}

based on the above information, if you can directly answer the user question, please provide the answer directly starting with "output" and providing a detailed answer.
if you have answered the question, then skip the next steps.
if you think any python code is required to answer the question, provide the python code to evaluate the answer, starting with "python" keyword. 
generate a complete executable python code. the format of the python code is something similar to this:
"def generate_answer(df_result):
    #do some analysis
    return answer_string
generate_answer"
here you have a function that generates the answer as a string, and returns that string. In addition, the name of the function is being added at the end.
generate a python code in the above format. make the generated answer string descriptive.
`;

  const code_res = await llm.call(prompt_str);
  // Remove the quotes and 'Python' keyword
  if (code_res.startsWith("output") || code_res.startswith("Output")) {
    const output_keyword = "output";
    const output = code_res.substring(output_keyword.length + 2); //adding 2 to exclude colon and space
    const user_query = "***Here is the answer to the above question:***";
    display_markdown(user_query);
    answer = "## " + output;
    display_markdown(answer);
  } else {
    console.log("output generated. here is the code:");
    console.log(code_res);
    console.log("printed python code");
    // Remove the quotes and 'Python' keyword
    const code = code_res.replace("python", "").replace(/`{3}/g, "");
    console.log(code);
    let coderun = pyodide.runPython(code);
  }
};
```

# Examples

```javascript
await chat("how many customers live in USA? ");
```

```diff
User Query:
how many customers live in USA?
helper steps to answer the question:

sql query generated. here is the query:

*SELECT COUNT() FROM customers WHERE Country = 'USA';

sqlite database downloaded

Loading micropip, packaging
Loaded micropip, packaging
Loading pandas, numpy, python-dateutil, six, pytz
Loaded numpy, pandas, python-dateutil, pytz, six
Loading sqlalchemy, sqlite3, typing-extensions
Loaded sqlalchemy, sqlite3, typing-extensions
data fetched
extracted data from the database. here is the data:

{"COUNT(*)":{"0":13}}
Here is the answer to the above question:

There are 13 customers who live in the USA.
```
