<fs x-large>Alexa Custom Skill - Stock Prices</fs>

-----------------------------------------------------------------------------------------------------------------------------

**Author:** Tristan Brodeur 

**Email:** brodeurtristan@gmail.com

**Date:** Last modified on 01/16/17

**Keywords:** Alexa, Amazon Skills Kit</fs>

-----------------------------------------------------------------------------------------------------------------------------

<fs large>Overview</fs>

Create an Alexa Skill that returns stock data for 10 listed companies.


-----------------------------------------------------------------------------------------------------------------------------

1. <fs large>Create a Lambda Function</fs>

To create a lambda function, follow this tutorial: [[using_lambda|Creating a lambda function]]

-----------------------------------------------------------------------------------------------------------------------------

2. <fs large>Add the code</fs>

******The code below is commented out in order to provide you with a better understanding. If you have any further questions, please feel free to contact me at the email listed above.******</fc>

```python
import requests
from bs4 import BeautifulSoup
'''
Amazon alexa skill to grab stock data from yahoo finance and present data to user
Currently has database of ~10 companies
Future updates to include database of all S&P 500 companies if data size allows
'''
##################################################################
def lambda_handler(event, context):
	#Checks to make sure application id is same as alexa skill (links)
	if (event['session']['application']['applicationId'] !=
		#Change to application id listed under your alexa skill in the developer portal
		"amzn1.ask.skill.XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"):
		raise ValueError("Invalid Application ID")

	if event["session"]["new"]:
		on_session_started({"requestId": event["request"]["requestId"]}, event["session"])
		#check event types and call appropriate response
		if event["request"]["type"] == "LaunchRequest":
			return on_launch(event["request"], event["session"])
		elif event["request"]["type"] == "IntentRequest":
			return on_intent(event["request"], event["session"])
		elif event["request"]["type"] == "SessionEndedRequest":
			return on_session_ended(event["request"], event["session"])
##################################################################
def on_session_started(session_started_request, session):
	print "Starting new."
##################################################################
def on_launch(launch_request, session):
	return get_welcome_response()
##################################################################
def on_intent(intent_request, session):
	intent = intent_request["intent"]
	intent_name = intent_request["intent"]["name"]

	if intent_name == "GetCurrentPrice":
		return get_current_price(intent)
	elif intent_name == "GetHigh":
		return get_high(intent)
	elif intent_name == "GetLow":
		return get_low(intent)
	elif intent_name == "AMAZON.HelpIntent":
		return get_welcome_response()
	elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
		return handle_session_end_request()
	else:
		raise ValueError("Invalid intent")
##################################################################
def on_session_ended(session_ended_request, session):
	print "Ending session."
##################################################################
def handle_session_end_request():
	card_title = "Stocker"
	speech_output = "Im sorry dave. Im afraid I cant do that."
	should_end_session = True
	return build_response({}, build_speechlet_response(card_title, speech_output, None, should_end_session))
##################################################################
def get_welcome_response():
	#Response to user invoking stock app without parameters/intents
	session_attributes = {}
	card_title = "Stocker"
	speech_output = "Welcome to the Stocker skill. " \
	 	"You can ask me for current stock prices, or " \
	 	"ask me about stock highs and lows."
	reprompt_text = "Please ask me for stock prices, " \
		"for example Apple stock price."
	should_end_session = False
	return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))

##################################################################
def find_availability(to_find, restaurant):
	page = requests.get("")
	page_data = BeautifulSoup(page.content, "html.parser")


##################################################################
def grab_data(to_find, ticker_code):

	if ticker_code != "EVRGRN":
		#send http response fro yahoo finance to grab data based on passed in ticker code
		page = requests.get("https://finance.yahoo.com/quote/" + ticker_code + "?p=" + ticker_code)
		#resolves the page content into its component parts
		data = BeautifulSoup(page.content, "html.parser")

		if to_find == "get_current_price":
			#grab text from appropriate class and return to current object
			current = data.find(class_ = "Fw(b) Fz(36px) Mb(-4px)").get_text()
			#find index of decimal point to seperate dollars from cents
			index = current.index('.')
			dollars = current[:index]
			cents =  current[index+1:]
			#map each component in list to a string object, and join via str operator (in this case nothing, so it joins together)
			dollars = ''.join(map(str, dollars))
			cents = ''.join(map(str, cents))
			return dollars + " dollars and " + cents + " cents"

		elif to_find == "get_high" or to_find =="get_low":
			classes= data.find_all(class_ = "Ta(end) Fw(b)")
			#several stock values are listed under the class with the given value, so
			#we grab fourth object in list of classes, returning stock range
			stock_range = list(classes[4].get_text())
			#since text we grab is not in unicode, we grab each str object and encode it with UTF8
			stock_range = [x.encode('UTF8') for x in stock_range]
			if to_find == "get_high":
				high = ""
				for letter in stock_range:
					#since returned list of str objects are a range from low to high (e.g. $$ - $$), we grab second half of data for high value
					if letter == '-':
						high = stock_range[stock_range.index(letter)+2:]
						index = high.index('.')
						dollars = high[:index]
						cents =  high[index+1:]
						#map each component in list to a string object, and join via str operator (in this case nothing, so it joins together)
						dollars = ''.join(map(str, dollars))
						cents = ''.join(map(str, cents))
						return dollars + " dollars and" + cents + " cents"
			elif to_find == "get_low":
				low = ""
				for letter in stock_range:
					#since returned list of str objects are a range from high to low (e.g. $$-$$), we grab first half of data for low value
					if letter == '-':
						low = stock_range[:stock_range.index(letter)-1]
						index = low.index('.')
						dollars = low[:index]
						cents =  low[index+1:]
						#map each component in list to a string object, and join via str operator (in this case nothing, so it joins together)
						dollars = ''.join(map(str, dollars))
						cents = ''.join(map(str, cents))
						return dollars + " dollars and " + cents + " cents"
	#cor is on its way
	else:
		if to_find == "get_current_price":
			return "over 9000"
		elif to_find == "get_high" or to_find =="get_low":
			return "not yet available. Please ask Tristan"
##################################################################
def get_current_price(intent):
	session_attributes = {}
	card_title = "Current Stock Price"
	speech_output = "I'm not sure which company you wanted stock prices for. " \
                    "Please try again."
	reprompt_text = "I'm not sure which company you wanted stock prices for. " \
                    "Please try again."
	should_end_session = False

	if "Ticker" in intent["slots"]:
    		ticker_name = intent["slots"]["Ticker"]["value"]
    		#grab appropriate value based on name from ticke code dict
        	ticker_code = get_ticker_code(ticker_name.lower())

      	if (ticker_code != "unkn"):
      		card_title = "Current price for  " + ticker_name.title()
      		price = grab_data(get_current_price.__name__, ticker_code)
      		speech_output = "Current price for  " + ticker_name + " is: " + price
      		reprompt_text = ""
      		should_end_session = True

	return build_response(session_attributes, build_speechlet_response(
        	card_title, speech_output, reprompt_text, should_end_session))
##################################################################
def get_high(intent):
	session_attributes = {}
	card_title = "Stock High"
	speech_output = "I'm not sure which company you wanted stock prices for. " \
                    "Please try again."
	reprompt_text = "I'm not sure which company you wanted stock prices for. " \
                    "Please try again."
	should_end_session = False

	if "Ticker" in intent["slots"]:
    		ticker_name = intent["slots"]["Ticker"]["value"]
    		#grab appropriate value based on name from ticke code dict
        	ticker_code = get_ticker_code(ticker_name.lower())

      	if (ticker_code != "unkn"):
      		card_title = "High price for  " + ticker_name.title()
      		price = grab_data(get_high.__name__, ticker_code)
      		speech_output = "Stock high for " + ticker_name +  " is: " + price
      		reprompt_text = ""
      		should_end_session = True

	return build_response(session_attributes, build_speechlet_response(
        	card_title, speech_output, reprompt_text, should_end_session))			
##################################################################
def get_low(intent):
	session_attributes = {}
	card_title = "Stock Low"
	speech_output = "I'm not sure which company you wanted stock prices for. " \
			"Please try again."
	reprompt_text = "I'm not sure which company you wanted stock prices for. " \
			"Please try again."
	should_end_session = False
	
	if "Ticker" in intent["slots"]:
		ticker_name = intent["slots"]["Ticker"]["value"]
		#grab appropriate value based on name from ticke code dict
		ticker_code = get_ticker_code(ticker_name.lower())
	
		if (ticker_code != "unkn"):
			card_title = "Low price for  " + ticker_name.title()
      		price = grab_data(get_low.__name__, ticker_code)
      		speech_output = "Stock low for " + ticker_name + " is: " + price
      		reprompt_text = ""
      		should_end_session = True
      		
	return build_response(session_attributes, build_speechlet_response(
      	card_title, speech_output, reprompt_text, should_end_session))
##################################################################
def build_speechlet_response(title, output, reprompt_text, should_end_session):
	#return data in json format to alexa skills kit
	return {
		"outputSpeech": {
     			"type": "PlainText",
     			"text": output
     		},
     		"card": {
     			"type": "Simple",
     			"title": title,
     			"content": output
     		},
     		"reprompt": {
     			"outputSpeech": {
     				"type": "PlainText",
     				"text": reprompt_text
     			}
     		},
     		"shouldEndSession": should_end_session
     	}
##################################################################
def get_ticker_code(ticker_name):
	#function to grab associated value based on ticker_name parameter
	return{
		'alphabet': 'GOOGL', 
		'apple': 'AAPL',
		'amazon':'AMZN', 
		'tesla':'TSLA', 
		'oracle':'ORCL', 
		'irobot':'IRBT',
		'microsoft':'MSFT', 
		'facebook':'FB',
		'intel':'INTC',
		'ibm':'IBM', 
		'evergreen robotics': 'EVRGRN',
		'netflix':'NFLX'
	}.get(ticker_name, "unkn")
##################################################################
def build_response(session_attributes, speechlet_response):
	return {
		"version": "1.0",
		"sessionAttributes": session_attributes,
		"response": speechlet_response
}		
```

******Make sure to change the application id listed in lambda_handler() to your own id once you create the Alexa Skill with Alexa Skills Kit******</fc>

-----------------------------------------------------------------------------------------------------------------------------

After the code is added, [configure your lambda function with the Alexa Skills Kit.](https://github.com/twbot/CreatingAnAlexaSkill)