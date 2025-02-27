# Computing with a Lua Window
## Overview
This example demonstrates how you can use Lua code to modify user data. The Lua window in this example is used as an alternative to a Compute window.

---
**NOTE:**
Use this example with SAS Event Stream Processing 2021.2.1 and later. 

---

## Use Case

The model handles customer information for a telecommunications company. A Lua window called TransformData is configured to perform several actions:

- Capitalize each customer’s surname.
- Determine each customer’s preferred method of communication and save this to a new field, `notify`.
  - If the customer has a telephone number, then set the preferred method of communication to `sms`.
  - If the customer does not have a telephone number but does have an email address, then set the preferred method of communication to `email`.
- Convert each telephone number to a standard format by reading in the final ten digits and removing all numerical characters.

## Source Data
The file [lua_compute.csv](lua_compute.csv) contains customer information.

## Workflow
The following figure shows the diagram of the project:

![Diagram of the project](https://github.com/sassoftware/esp-studio-examples/blob/main/Basic/lua_compute/img/lua-compute-window-example2.png "Diagram of the project")

### Customers

The Customers window streams customer information from the lua_compute.csv file to the TransformData window.

Explore the settings for the Customers window:
1. Open the project in SAS Event Stream Processing Studio and select the Customers window. 
2. To examine the window's output schema, on the right toolbar, click ![Output Schema](https://github.com/sassoftware/esp-studio-examples/blob/main/Basic/lua_compute/img/output-schema-icon.png "Output Schema"). Observe the following fields: 
   - `custref`: This is the customer reference ID. It is also selected as the Key.
   - `forename`: This is the customer’s first name.
   - `surname`: This is the customer’s last name.
   - `telno`: This is the customer’s telephone number.
   - `email`: This is the customer’s email address.
3. Click ![Properties](https://github.com/sassoftware/esp-studio-examples/blob/main/Basic/lua_compute/img/show-properties-icon.png "Properties"). 

### TransformData

The TransformData window is configured to perform several actions. For more information, see [Use Case](#use-case).

Explore the settings for this window:
1. Select the TransformData window in the workspace.
2. In the right pane, expand **Lua Settings**.
   - To increase the efficiency of the Lua code processing, you can specify which fields to copy from the source file and which fields to use in the Lua code. Note the following details in this example:
     - The **Fields to copy** field shows that the fields `surname` and `telno` are excluded. All other fields from the source file can be copied because they are not transformed by the Lua code.
     - The **Fields to use in Lua code** field shows that the fields `surname` and `telno` are included. These fields are transformed by the Lua code.
   - Observe that the **Events function** field shows the Lua function that is used (in this case, the `create` function).
3. Scroll down in the right pane, to view the Lua code that performs the calculations for this example:
   
    <table>
    <tr>
    <th>Step</th> <th>Lua Code Section</th>
    </tr>
    <tr>
    <td>Read in the data from the lua_compute.csv file.</td>
    <td>

      
    ```
    function create(data,context)

      local e = {}
    ```

      
    </td>
    </tr>
    <tr>
    <td> Capitalize each customer’s surname.</td>
    <td>


    ```
      e.surname = string.upper(data.surname)
    ```


    </td>
    </tr>
    <tr>
    <td> Retain the final ten digits of each telephone number.</td>
    <td>


    ```
      e.telno = cleanNumber(data.telno)
    ```


    </td>
    </tr>
    <tr>
    <td>If a customer has a telephone number, then assign the value sms.

    </td>
    <td>


    ```
      if (e.telno ~= "")
      then
        local len = string.len(e.telno)
        if (len > 10)
        then
          e.telno = string.sub(e.telno,len - 10 + 1)
        end
        e.notify = "sms"
    ```


    </td>
    </tr>
    <tr>
    <td>Otherwise, if a customer does not have telephone number, then assign the value email.</td>
    <td>


    ```
      else
        e.notify = "email" -- 5
      end

      return(e)
    end
    ```


    </td>
    </tr>
    <tr>
    <td>To bring the telephone numbers into a standardized format, remove all non-alphanumeric characters from each telephone number.</td>
    <td>


    ```
    function cleanNumber(telno)
      local   str = ""
      
      if (telno ~= "" and telno ~= nil)
      then
        str = ""
        for c in telno:gmatch("[0-9]")
        do
          str = str..c
        end
      end

      return(str)
    end
    ```
    </td>
    </tr>
    </table>


## Test the Project and View the Results

When you test the project in SAS Event Stream Processing Studio, you can use the **Publish** button to publish events from the lua_compute.csv file into the Customers window. For more information, see [Publish Events from a CSV file](https://go.documentation.sas.com/doc/en/espcdc/default/espstudio/p124n2fohetwqzn109gsdel6o1cj.htm).

Alternatively, before you test the project, you can configure a publisher connector for the Customers window. For more information, see [Configure a File and Socket Connector](https://go.documentation.sas.com/doc/en/espcdc/default/espstudio/n0esv2n0cbbpgcn1r281krr1iv6q.htm#n0y87cwr7q5vo6n1qlfcey182vt6).

The results for each window appear in separate tabs in test mode. The following figure shows the customer data in the **Customers** tab. Note the following details:
- Each customer’s surname is in title case.
- The telephone numbers were not entered using the same format.
- The customers Hermione Grandeur and Draco Molefay have no entry for `telno`.

![Results for the Customers tab](https://github.com/sassoftware/esp-studio-examples/blob/main/Basic/lua_compute/img/Customers.png "Results for the Customers tab")

The following figure shows the transformed data in the **TransformData** tab. Note the following changes:
- All surnames are now in all capital letters.
- Each telephone number shows ten digits with no spaces or non-numerical characters.
- A new variable, `notify`, has been created. Each customer with a telephone number has been assigned the value `sms`, whereas the customers without a telephone number (Hermione Grandeur and Draco Molefay) have been assigned the value `email`.

![Results for the TransformData tab](https://github.com/sassoftware/esp-studio-examples/blob/main/Basic/lua_compute/img/TransformData.png "Results for the TransformData tab")

## Additional Resources
For more information, see [SAS Help Center: Using Lua Windows](https://documentation.sas.com/?cdcId=espcdc&cdcVersion=default&docsetId=espcreatewindows&docsetTarget=p0yj92wgv3ssyyn1syatsh9l1t74.htm).
