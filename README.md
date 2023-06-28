### Prerequisites (Your own machine)
This playground uses the following technology so you will need to check they are installed:

-   VS Code
-   Python Extension for VSCode
-   Terminal (We use Bash)
-   Python
-   Flask
-   Pytest
-   Some knowledge of basic unit testing
-   Some knowledge of programming fundamenals

# Introduction
Test Driven Development is a software development methodology where tests are written before application code is written.
The idea is that a developer will think about how the user will use the application/components through the process of writing
tests that validate the operation.

In this playground we are going to build a web application in python and Flask via the TDD process.
We will write tests to describe the functionality of the components.

# set up your environment

### Opening the Next.js application in VS Code

### Step 1

Go back to https://lab.devopsplayground.org/.

If you need to, enter your name and surname again into the username box. 

You should be presented with the following page: 

<p align="center">
<img width="544" alt="Screenshot 2023-05-23 at 11 33 12" src="https://github.com/DevOpsPlayground/DPG-Meetups-Next.js/assets/101208108/a8e6b536-6926-4ae6-a72a-5c19e7f4d207">
</p>

### Step 2

Copy the IDE link

Open a new tab and paste that link into the address bar of the new tab. 

You should be presented with VS Code which should look something like this:


### Step 3
Make sure you have the python extension installed in .vscode which is automatically generated by the IDE.

We will be using the Testing toolbar in vscode. For this project you will need to manually change the .vscode that is auto generated by your IDE.
<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/settings.png">
</p>

in the settings.json, replace the contents "python.testing.pytestArgs" with:

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/ide_view.png">
</p>


```json
    "python.testing.pytestArgs": [
        "project/tdd_stock/test"
    ],
```

### 4
In project/tdd_stock/test click on test_unit.py.
This should activate the test tool bar in the IDE on the left (you may need to restart the browser)
When you click on the test tool bar you should be able to see the test tree.

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/test_tree.png">
</p>


# Code base

App: the application directory where the flask application will be in

- app.py: the main flask application with the routes
- __init__.py : initialisation file for python.
- components.py: file containing helper functions for app.py
- models.py: contains the Stock class which defines the model for a stock

db: database file for our application

- stock_db.json: the database file containing stocks based off the Stock model.
This file will act as our database for our application

test: Directory for tests and tes resources

- __init__.py : initialisation file for python.
- test_data/stock_test.json: a test stock object that will be used in our tests
- notes.txt: notes contianing curl commands for manual testing
- test_int.py: integration tests will be written here
- test_unit.py: unit tests will be written here

# The Playground

### Step 1

We want to implement a feature where we can get all the stocks from the database as a list of stock objects.

Go to project/tdd_stock/test/test_unit.py. You will see test_get_all_stocks as the first test.

```python

    def test_get_all_stocks_returns_all(self):
        stocks = get_all_stocks()

```

This test has not been fully implemented. So far it calls the get_all_stocks function but there are no assertions. Lets add a simple assertion to get us started.
We know that the database file has 3 stock objects. As a simple assertion we can check that when we get all stocks the list should be a length of 3.
Add this assertion to the test function

```python

    def test_get_all_stocks_returns_all(self):
        stocks = get_all_stocks()
        
        assert len(stocks) == 3

```

If you run the test it will fail

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/get_all_stocks_initial_error.png">
</p>

Unsuprisingly, this is because the test function has no visibility of get_all_stocks. Since a stub function is already present in components.py, lets import it at the top of the test file. You can put it underneath the Stock model import

```python

from app.models import Stock
from app.components import get_all_stocks

```
If you rerun the test you now get this error:

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/get_all_stocks_len_error.png">
</p>

We have now reached the "red" stage of the red, green, refactor TDD flow. We can fix this error by now implementing some application code to make the test pass.

### Step 2

Go to app/components.py and see the get_all_stocks_function

```python
def get_all_stocks():

    with open('project/tdd_stock/db/stock_db.json') as dbfile:
        stocks_json = json.load(dbfile)
```

The function so far reads the db file and loads it into a json list using pythons json library.
Since the DB file is in the form of a json list, json.load will produce a list.
Lets return the list

```python
def get_all_stocks():

    with open('project/tdd_stock/db/stock_db.json') as dbfile:
        stocks_json = json.load(dbfile)

    return stocks_json
```

The test will now pass, the "Green" stage in TDD.

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/get_all_stocks_test_pass_initial.png">
</p>

### Step 3

Now that we have a passing test lets go into the "Refactor" stage of TDDs red, green, refactor. Part of the requirement for this component is that a list of stock objects is returned. Our test so far only checks the length of the list. Lets add an assertion to prove that we ge back a list of stocks.

Go to project/tdd_stock/test/test_unit.py

```python
    def test_get_all_stocks_returns_all(self):
        stocks = get_all_stocks()
        assert len(stocks) ==3
        assert stocks[0].analyst == 'warren.buffet'
```
 By invoking the analyst variable we are proving that the first element in the list is a Stock object that has the attribute 'analyst'

 After rerunning the test you should get an error like:

```
 Failed: [undefined]AttributeError: 'dict' object has no attribute 'analyst'
```
This is because get_all_stocks so far only returns a list of pythin dicitionaries, not a list of Stocks. Lets refactor our function to support that.

Go to project/tdd_stock/app/components.py. Update the function with a lambda expression that will iterate through the list of dictioanries and convert them
to Stock objects

```python
def get_all_stocks():

    with open('project/tdd_stock/db/stock_db.json') as dbfile:
        stocks_json = json.load(dbfile)

    stocks_list_obj = list(map(lambda x: Stock(**x), stocks_json))

    return stocks_list_obj
```

The test will now pass once rerun.

### Step 4

We have implemented a function, now lets use it in our app.py and associate it with a route. We want the route to return our json dictionary.

Go to app/app.py uncomment the function with the rout '@app.route("/stock/all_stocks/", methods=["GET"])'

```python
    @app.route("/stock/all_stocks/", methods=["GET"])
    def get_all():
        stocks = get_all_stocks()
        return jsonify([vars(e) for e in stocks])
```

We can run this integration test by going to the test tree

<p align="center">
<img width="668" alt="Screenshot 2023-05-23 at 10 06 27" src="https://github.com/cgungaloo/tdd_stock/blob/master/readme_resources/get_all_stock_integration_tree.png">
</p>

Running this will give the same error as the unit test 'Failed: [undefined]NameError: name 'get_all_stocks' is not defined' we can fix this by doing an import in

in app/app.py add the import

```python
from app.components import get_all_stocks
```

The test will now pass.

In this test we are asserting that we get a 200 response code from the endpoint. We can add a another assertion by checking the length of the response.

in project/tdd_stock/test/test_int.py add to the test_get_all_stocks

```python
    def test_get_all_stocks(self):
        response = self.client.get("/stock/all_stocks/")

        assert response.status_code == 200
        stock_json = response.json
        assert len(stock_json) == 3
```

If you want you can add some more assertions too.

We have now implemented a function and a route via TDD!

### Step 5

Lets implement the next function, get_stock_by_ticker.

Go to project/tdd_stock/test/test_unit.py

uncomment the function 'def test_get_stock_by_ticker_returns_correct_stock(self):'

```python
    def test_get_stock_by_ticker_returns_correct_stock(self):
        pass
```

this function will return a single stock object when a ticker symbol is passed as a string.
Lets implement the test to define how the application function will work.

```python
    def test_get_stock_by_ticker_returns_correct_stock(self):
        
        stock = get_stock_by_ticker('MSFT')

        assert stock.ticker_symbol == 'MSFT'
        assert stock.name == 'Microsoft'

```

lets also import get_stock_by_ticker

```python
from app.components import get_all_stocks, get_stock_by_ticker
```
We have now defined a test that we can implement against.

If you run the test now, you will get: 'test_get_stock_by_ticker_returns_correct_stock Failed: [undefined]AttributeError: 'NoneType' object has no attribute 'ticker''

Lets make this test go green. In project/tdd_stock/app/components.py our function looks like:

```python
def get_stock_by_ticker(ticker_symbol):
    with open('project/tdd_stock/db/stock_db.json') as dbfile:
        stocks_json = json.load(dbfile)
        
    pass
```

We need to read the stocks and find the one that matches the ticker symbol. We will use a filter function to find the stock we want

```python

def get_stock_by_ticker(ticker_symbol):
    with open('project/tdd_stock/db/stock_db.json') as dbfile:
        stocks_json = json.load(dbfile)

    stocks_list_obj = list(map(lambda x: Stock(**x), stocks_json))

    stock_list_match = list(filter(lambda x:x.ticker_symbol == ticker_symbol, stocks_list_obj))

    return stock_list_match[0]

```

After rerunning the test, it will pass.

You may notice that some of the code has been repeated from get_all_stocks. We can now do a refactor to make th function abit simpler by calling get_all_stocks

```python
def get_stock_by_ticker(ticker_symbol):
    stocks_list_obj = get_all_stocks()

    stock_list_match = list(filter(lambda x:x.ticker_symbol == ticker_symbol, stocks_list_obj))

    return stock_list_match[0]
```

We have reduced the code by using the get_all_stocks function implemented earlier.
If you re run all the tests that we have written so far, they should all still pass.

### Step 6

Lets now use get_stock_by_ticker in app/app.py

uncomment the function with the route '@app.route("/stock/<ticker_symbol>/",methods=["GET"])'

```python
@app.route("/stock/<ticker_symbol>/",methods=["GET"])
def get_stock_by_ticker_symbol(ticker_symbol):
    
    # if stock:
    return jsonify(stock.__dict__)
    # else:
    #     return jsonify(None)

```

in project/tdd_stock/test/test_int.py uncomment 'test_get_stock_by_ticker_integration'

This test has been implemented, but if you run it, it will fail: 'test_get_stock_by_ticker_integration Failed: [undefined]NameError: name 'stock' is not defined'

Lets make the test pass by fixing the app function.
The function fails because stock is undefined. In app/app.py you can see that get_stock_by_ticker is not being called. Lets call it.

```python
@app.route("/stock/<ticker_symbol>/",methods=["GET"])
def get_stock_by_ticker_symbol(ticker_symbol):
    
    stock = get_stock_by_ticker(ticker_symbol)
    # if stock:
    return jsonify(stock.__dict__)
    # else:
    #     return jsonify(None)

```

Make sure you import get_stock_by_ticker

```python
from app.components import get_all_stocks, get_stock_by_ticker

```

The commented if statement will be used later in other functionality.
The tests will now pass.

### Step 7

What if a ticker_symbol we provide is not found in the DB? Lets write a test to describe what the expected behaviour should be

uncomment 'test_invalid_stock_not_found' in project/tdd_stock/test/test_unit.py

```python
    def test_invalid_stock_not_found(self):
        stock = get_stock_by_ticker("TSLA")
        assert stock == None
```

This test checks that is we pass an invaldi ticker symbol we get None returned.
This test when run, fails.

```
Failed: [undefined]IndexError: list index out of range
self = <test.test_unit.StockTestClass testMethod=test_invalid_stock_not_found>
```

Lets make this test green by fixing the function under test, get_stock_by_ticker

```python
def get_stock_by_ticker(ticker_symbol):
    stocks_list_obj = get_all_stocks()
    
    stock_list_match = list(filter(lambda x:x.ticker_symbol == ticker_symbol, stocks_list_obj))

    if len(stock_list_match) ==0:
        return None
    return stock_list_match[0]
```

The test now passes

### Step 8

We can now write a test to see how this function is used within a route.

Go to project/tdd_stock/test/test_int.py uncomment the test_get_stock_by_bad_ticker_integration test

```python
    def test_get_stock_by_bad_ticker_integration(self):

        response = self.client.get(
            f"/stock/TSLA/",
            content_type="application/json"
        )
```

We expect this route to return a 200 response and the json conent to be None when making a call with an invalid ticker_symbol

```python
    def test_get_stock_by_bad_ticker_integration(self):

        response = self.client.get(
            f"/stock/TSLA/",
            content_type="application/json"
        )

        assert response.status_code == 200
        assert response.json == None
```
We can pass this failing test by uncommenting the if statement in

```python
@app.route("/stock/<ticker_symbol>/",methods=["GET"])
def get_stock_by_ticker_symbol(ticker_symbol):
    
    stock = get_stock_by_ticker(ticker_symbol)
    if stock:
        return jsonify(stock.__dict__)
    else:
        return jsonify(None)

```

This test will now pass

### Step 9

The next requirement is to add a stock to the DB. Lets think about how to test that.

Go to test/test_unit.py

uncomment test_save_stock_success

```python
    def test_save_stock_success(self):

    with open('project/tdd_stock/db/stock_db.json) as f:
        stock_data = json.load(f)
```

You'll notice that there is a reference to 'project/tdd_stock/db/stock_db.json' this is a test file which contains a stock object. We will use this to update the db.

We can know if save_stock works by shceking the state of the DB after upload. if we call get_all_stocks after save_stock, we expect the lentgh of the DB to now be 4.

Update the test

```python
    def test_save_stock_success(self):

        with open('project/tdd_stock/test/test_data/stock_test.json') as f:
            stock_data = json.load(f)

        save_stock(stock_data)

        stocks = get_all_stocks()

        assert len(stocks) == 4
```

and import save_stock

```python
from app.components import get_all_stocks, get_stock_by_ticker, save_stock
```

in project/tdd_stock/app/components.py uncomment save_stock

```python
def save_stock(stock_to_save):
    with open('project/tdd_stock/db/stock_db.json','r') as json_db:
        stock_list = json.load(json_db)
    # stock_list_obj = list(map(lambda x:Stock(**x), stock_list))
    # stock_obj = Stock(**stock_to_save)
    # stock_list_obj.append(stock_obj)

    stock_list_json = list(map(lambda x: vars(x), stock_list_obj))
    with open('db/stock_db.json','w') as json_db:
        json.dump(stock_list_json,json_db,sort_keys=True, indent=4, separators=(',', ': '))
    # else:
    #     raise Exception(f"{stock_obj.ticker_symbol} already exists")
```

Running the test gives this error: 'test_save_stock_success Failed: [undefined]TypeError: app.models.Stock() argument after ** must be a mapping, not list'

We are now at Red stage in Red, Green, Refactor. Lets pass the test.
We need to get all stocks and convert it to a list of stocks. We then take the incoming stock and add it to the list. We then rewrite the updated list to the DB

uncomment the code that does this

```python
def save_stock(stock_to_save):
    with open('project/tdd_stock/db/stock_db.json','r') as json_db:
        stock_list = json.load(json_db)
    stock_list_obj = list(map(lambda x:Stock(**x), stock_list))

    stock_obj = Stock(**stock_to_save)

    # if get_stock_by_ticker(stock_obj.ticker_symbol):
    #     raise Exception(f"{stock_obj.ticker_symbol} already exists")
    # else:
    stock_list_obj.append(stock_obj)

    stock_list_json = list(map(lambda x: vars(x), stock_list_obj))
    with open('project/tdd_stock/db/stock_db.json','w') as json_db:
        json.dump(stock_list_json,json_db,sort_keys=True, indent=4, separators=(',', ': '))
```

The test should now pass. We can now add a further assertion to check that the new stock is added.

```python
    def test_save_stock_success(self):

        with open('project/tdd_stock/test/test_data/stock_test.json') as f:
            stock_data = json.load(f)

        save_stock(stock_data)

        stocks = get_all_stocks()

        assert len(stocks) == 4
        assert stocks[3].ticker_symbol == 'HTHIY'
```

This proves that the last entry in the DB was stock from the test data.
If you go to the DB file, you'll notice that you can't see the new stock in the file. This is because the unit test has a teadDown function that automatically removes the entry after the test. This allows you to run the test repeatedly without it growing the DB. This will become important when we implement a duplication check later

### Step 10

Lets now use this function in a route.

Go to project/tdd_stock/test/test_int.py. Uncomment test_add_stock_integration

```python
    def test_add_stock_integration(self):

        with open('test/test_data/stock_test.json') as f:
            stock_data = json.load(f)

        data_json = json.dumps(stock_data)
```

At a route level we want to check that the function returns a 200 response when we call the endpoint.


```python

    def test_add_stock_integration(self):

        with open('project/tdd_stock/test/test_data/stock_test.json') as f:
            stock_data = json.load(f)

        data_json = json.dumps(stock_data)

        response = self.client.post(
            f"/add-stock/",
            content_type="application/json"
            ,data = data_json
        )

        assert response.status_code == 200
```

Uncomment @app.route("/add-stock/",methods=["POST"]) in app/app.py

```python
@app.route("/add-stock/",methods=["POST"])
def add_stock_to_db():
    # try catch for error
    # try:
    save_stock(request.json)
    resp = jsonify(success=True)
    return resp
    # except Exception as e:
    #     return Response(f'{str(e)}',status=400)
```

Import the function too

```python
from app.components import get_all_stocks, get_stock_by_ticker, save_stock

```
The test will now pass.

### Step 10

If the DB already has a ticker_symbol that we want to add, we want our application to reject it. Lets write a test for this.

Go to test/test_unit.py. Uncomment test_save_duplicate_stock_rejected

```python
    def test_save_duplicate_stock_rejected(self):
        prices = [
            {
                "date": "2022-01-01",
                "value": 201
            },
            {
                "date": "2022-01-02",
                "value": 199
            },
            {
                "date": "2022-01-03",
                "value": 205
            },
            {
                "date": "2022-01-04",
                "value": 205
            },
            {
                "date": "2022-01-05",
                "value": 206
            }
        ]

        # create stock and convert to __dict__

        # with pytest.raises(Exception, match="MSFT already exists") as excp:
        #     save_stock(data_json)
```

The prices variable is a list of dictioanries. each key value pair is a date and price. We will use prices in the creation of a new stock.

```python
        # create stock and convert to __dict__
        stock = Stock(analyst='bill.gates', name='Microsoft', ticker_symbol='MSFT', prices=prices)
        data_json = stock.__dict__
```

We then want to catch an exception and assert the correct error message. Uncomment this part

```python
    with pytest.raises(Exception, match="MSFT already exists") as excp:
        save_stock(data_json)
```

Running this test will fail with this error message

'''
test_save_duplicate_stock_rejected Failed: [undefined]Failed: DID NOT RAISE <class 'Exception'>
'''

Lets go to our add_stock function and fix that. Go to project/tdd_stock/app/components.py

We are going to need to implement an if statement that checks if a stock exists based on the ticker_symbol provided.


```python
def save_stock(stock_to_save):
    with open('project/tdd_stock/db/stock_db.json','r') as json_db:
        stock_list = json.load(json_db)
    stock_list_obj = list(map(lambda x:Stock(**x), stock_list))

    stock_obj = Stock(**stock_to_save)

    if get_stock_by_ticker(stock_obj.ticker_symbol):
        raise Exception(f"{stock_obj.ticker_symbol} already exists")
    else:
        stock_list_obj.append(stock_obj)

        stock_list_json = list(map(lambda x: vars(x), stock_list_obj))
        with open('project/tdd_stock/db/stock_db.json','w') as json_db:
            json.dump(stock_list_json,json_db,sort_keys=True, indent=4, separators=(',', ': '))
```

The if statment uses the previously implemented get_stock_by_ticker function. If None is return then we know the ticker_symbok is not in the DB. If however we do get an entry, the the stock already exists and we raise an exception.

Running the test again will pass.

### Step 11

Let do some refactoring on save_stock.

We can see that its opening the DB and converting it to a list of stocks. This was already implementted in the save_stock function. We can repalce our existing code with that.

```python
def save_stock(stock_to_save):

    stock_list_obj = get_all_stocks()
    stock_obj = Stock(**stock_to_save)

    if get_stock_by_ticker(stock_obj.ticker_symbol):
        raise Exception(f"{stock_obj.ticker_symbol} already exists")
    else:
        stock_list_obj.append(stock_obj)

        stock_list_json = list(map(lambda x: vars(x), stock_list_obj))
        with open('project/tdd_stock/db/stock_db.json','w') as json_db:
            json.dump(stock_list_json,json_db,sort_keys=True, indent=4, separators=(',', ': '))
            
```

we can also make the code abit more efficient by moving it to the else clause so that we only access the DB when we need to.

```python
def save_stock(stock_to_save):

    stock_obj = Stock(**stock_to_save)

    if get_stock_by_ticker(stock_obj.ticker_symbol):
        raise Exception(f"{stock_obj.ticker_symbol} already exists")
    else:
        stock_list_obj = get_all_stocks()
        stock_list_obj.append(stock_obj)

        stock_list_json = list(map(lambda x: vars(x), stock_list_obj))
        with open('project/tdd_stock/db/stock_db.json','w') as json_db:
            json.dump(stock_list_json,json_db,sort_keys=True, indent=4, separators=(',', ': '))   
```

Rerunning the tests will continue to pass. This is an example of how TDD can help you test as you go along and catch regressions sooner.

### Step 12

Lets implement an integration test for our duplication feature in project/tdd_stock/test/test_int.py. Uncomment test_add_stock_duplicate_rejected

```python

    def test_add_stock_duplicate_rejected(self):

        prices = [
            {
                "date": "2022-01-01",
                "value": 201
            },
            {
                "date": "2022-01-02",
                "value": 199
            },
            {
                "date": "2022-01-03",
                "value": 205
            },
            {
                "date": "2022-01-04",
                "value": 205
            },
            {
                "date": "2022-01-05",
                "value": 206
            }
        ]

        stock =  Stock("dave","Microsoft","MSFT",prices)
        data_json = stock.__dict__
        
        data_json = json.dumps(data_json)
        response = self.client.post(
            "/add-stock/",
            data = data_json,
            content_type = "application/json"
        )
```

We will add a response code check to our test.

```python
    assert response.status_code ==400
```

This failing test can be fixed in the route function found in project/tdd_stock/app/app.py

```python
@app.route("/add-stock/",methods=["POST"])
def add_stock_to_db():
    # try catch for error
    try:
        save_stock(request.json)
        resp = jsonify(success=True)
        return resp
    except Exception as e:
        return Response(f'{str(e)}',status=400)
```

The test now passes.

We can also add a further assertion to check the error message

```python
    assert response.data == b'MSFT already exists'
```


### Step 13

That last requirement to meet is to convert a stocks prices by a given currency code. The function will take a ticker symbol and a currency code. A Stock object is returned with the stock retrieved from the DB and the prices updated with the exchange rate for that currency code.

Go to project/tdd_stock/test/test_unit.py

uncomment test_convert_currency_is_correct

```python
    def test_convert_currency_is_correct(self):

        response = requests.get("https://open.er-api.com/v6/latest/USD")
        response = response.json()
        rates = response['rates']
        exchange_rate =rates['GBP']
        stock_expected = get_stock_by_ticker("APPL")

        # lambda with value
```

The starter code uses the requests library to make a get request to open.er-api.com api. This is an API that returns currency excahnge rates relative to USD.
We then go to the 'rates' dictionary and get the exchange rate for 'GBP'
We want to test that the stock returned from get_stock_with_conversion has the ticker_symbol 'APPL' and that the prices returned are what we calculate.

The first thing we need to do is generate our expected prices.

```python
    prices = stock_expected.prices

    expected_prices = list(map(lambda x:x['value'] * exchange_rate, prices))
```

We then want to call the fucntion and assert that expected_prices are the same as the actual prices returned.

```python
    stock_actual = get_stock_with_conversion('APPL', 'GBP')

    assert expected_prices == stock_actual.prices
    assert stock_actual.ticker_symbol == 'APPL'
```

In project/tdd_stock/app/components.py uncomment get_stock_with_conversion(ticker_symbol,conversion)

```python
    def get_stock_with_conversion(ticker_symbol,conversion):

        response = requests.get("https://open.er-api.com/v6/latest/USD")
        response = response.json()
        rates = response['rates']
        exchange_rate =rates[conversion]

        stock = get_stock_by_ticker(ticker_symbol)

        converted_prices = list(map(lambda x:x["value"]* exchange_rate,stock.prices))
        stock.prices = converted_prices
```

In project/tdd_stock/test/test_unit.py import the function

```python
from app.components import get_all_stocks, get_stock_by_ticker, save_stock, get_stock_with_conversion
```

This test returns the error

```
test_convert_currency_is_correct Failed: [undefined]AttributeError: 'NoneType' object has no attribute 'prices'
```

Lets go to our application code in project/tdd_stock/app/components.py
The function is not treturning anything. All we need to is return the stock

```python
return stock
```

the test will now pass

### Step 14

We will now write an integration test for the route using get_stock_with_conversion

```python
    def test_get_stock_by_ticker_conversion_integration(self):

        response = self.client.get(
            f"/stock/conversion/APPL/GBP",
            content_type="application/json"
        )

        json_response = response.json
        stock = Stock(**json_response)
        assert stock.ticker_symbol == 'APPL'
        assert response.status_code ==200
```

This test will fail as the route has not been implemented yet.
In project/tdd_stock/app/app.py uncomment @app.route("/stock/conversion/<ticker_symbol>/<conversion>", methods=["GET"])

```python
    @app.route("/stock/conversion/<ticker_symbol>/<conversion>", methods=["GET"])
    def get_stock_by_conversion(ticker_symbol,conversion):
        pass

```

We need to call the get_stock_with_conversion function return the response

```python
    @app.route("/stock/conversion/<ticker_symbol>/<conversion>", methods=["GET"])
    def get_stock_by_conversion(ticker_symbol,conversion):
        stock = get_stock_with_conversion(ticker_symbol,conversion)
        if stock:
            return jsonify(stock.__dict__)
        else:
            return jsonify(None)
```

The test now passes.

### Step 15

We can now start up the application on the server and try some curl commands.


```bash
    FLASK_APP=app/app.py python3 -m flask run
``` 
