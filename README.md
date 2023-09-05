## The task consists of the following parts:

- Recreate design and layout of the demo
- Add a &quot;Download&quot; button to the first of the 4 blocks that will download CSV data and display a
  loading animation (like in demo)
  ### CSV Data
  To get the parsed CSV data, you must merge data from Items API into Orders API, convert currency and
  format dates. The resulting array must be converted into a CSV string.
  Task notes:
- Orders API: https://api-staging.entriwise.com/mock/test-task-orders
- Items API: https://api-staging.entriwise.com/mock/test-task-items
- Convert amounts from specified currency to USD using https://api.exchangerate.host
- Convert YYYY-MM-DD to MMM dd
- Resulting CSV file must consist of 1001 lines
  Some fields in the dataset might be missing, mark those fields as N/A in CSV
  Expected CSV columns:
- Item Name
- Order ID
- Order Date
- Amount
  Implementation notes:
- Please solve the task below in **modern JavaScript / HTML / CSS and current best practices**
- Ensure the code is both **optimized and readable**
- Provide the amount of time it took you to finish it
  Please send your implementation as a link to https://jsfiddle.net/
