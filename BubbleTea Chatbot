import random
import math
import re
import datetime
from nltk.tokenize import RegexpTokenizer, word_tokenize
from nltk.metrics import edit_distance
import pgeocode
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

# Commented out elements with '#!' are used for testing purposes
# Global Variables represting menu items
drinks_menu = {
    'strawberry pineapple explosion': 4.30,
    'unicorn confetti': 3.75,
    'stormy pouf': 3.75,
    'original milk tea': 3.50,
    'sunshine yogurt': 4.50
    }

toppings_menu = {
    'tapioca pearls': 0.75,
    'grass jelly': 1.5,
    'red bean': 0.75,
    'whipped cream': 1.00,
    }

drink_strings = (" ".join(drinks_menu.keys()))
toppings_strings = (" ".join(toppings_menu.keys()))

size_menu = {
    'large': 1.00,
    'medium': 0.5,
    'small': 0
    }

specials = {
    0: 'strawberry pineapple explosion',
    1: 'unicorn confetti',
    2: 'stormy pouf',
    3: 'original milk tea',
    4: 'sunshine yogurt'
    }

day_of_the_week = datetime.datetime.today().weekday()
day_of_the_week_name = datetime.datetime.now().strftime("%a")
todays_special = specials[day_of_the_week]
drinks_menu[todays_special] -= 1.5 # represents the daily special discount
big_menu ={**size_menu, **toppings_menu, **drinks_menu}
# Global variables end here

# Haversine forumla for measuring distance between points
def distance(origin_postal):
    nomi = pgeocode.Nominatim('ca')
    lat1 = nomi.query_postal_code(origin_postal).latitude
    lon1 = nomi.query_postal_code(origin_postal).longitude
    lat2 = nomi.query_postal_code("m4y").latitude
    lon2 = nomi.query_postal_code("m4y").longitude
    radius = 6371 # km
    dlat = math.radians(lat2-lat1)
    dlon = math.radians(lon2-lon1)
    a = math.sin(dlat/2) * math.sin(dlat/2) + math.cos(math.radians(lat1)) \
        * math.cos(math.radians(lat2)) * math.sin(dlon/2) * math.sin(dlon/2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1-a))
    d = radius * c
    minutes = (d/30) * 60 #assume 30 kmph
    return round(minutes + 10) # for buffer

def menu_printer(which_menu):
    
    if which_menu == 'toppings':
        print("------------------------------------------------------")
        print("|Toppings                                    |Price  |")
        print("------------------------------------------------------")
        for key, value in toppings_menu.items():
            if key == "no": continue
            print(f"|{key}",' '*(42 -len(key)),f"|${value}",
                  ' '*(4 - len(str(value))),"|")
            
    if which_menu == "drinks":
        print("------------------------------------------------------")
        print("|Drinks (+$1.00 for Large, +$0.5 for Medium) |Price  |")
        print("------------------------------------------------------")
        for key, value in drinks_menu.items():
            if key == todays_special:
                key = key + f" ({day_of_the_week_name}'s Special)"
            print(f"|{key}",' '*(42 -len(key)),f"|${value}",
                  ' '*(4 - len(str(value))),"|")

class Drink:
    # Str, listof Str, Int, Int, Int
    def __init__(self, name=None, toppings=["nothing"], size=None, 
                 sugar=None, ice=None):
        if toppings is None:
            toppings = []
        self.toppings = toppings
        self.name = name
        self.toppings = toppings
        self.size = size
        self.sugar = sugar
        self.ice = ice
    
    def total_cost(self):
        cost_of_toppings = 0
        for toppings in self.toppings:
            if toppings == 'nothing': break
            cost_of_toppings += toppings_menu[toppings]
        total_cost = (drinks_menu[self.name] + 
                      size_menu[self.size] +
                      cost_of_toppings)
        return total_cost
    
    # used to inform the user of their drink
    def print_drink(self):
        toppings_as_string = ", ".join(self.toppings)
        print(f"""
        A {self.size} {self.name} with
        {self.sugar} sugar, {self.ice} ice, with {toppings_as_string} as toppings.""")
    

class BTBot:
    finished_responses = ["done", "finished", "finish", "that's it", 
                          "thats it", "all", "that is all", "no more", "no"]
    order_items = []
    helped = False # if we help them already, the chat phrase will be different
    exit_commands = ["quit", "pause", "exit", "goodbye", "bye", "later"]
    unable_to_communicate = 0 # 3 unables and the user is told to call
    last_reference = todays_special
    menu_words = ["menu", "price", "choice", "choices"]
    keywords = ["strawberry", "pineapple", "explosion", "unicorn", "confetti",
                "stormy", "pouf", "original", "milk", "tea", "sunshine", 
                "yogurt", "tapioca", "pearls", "grass", "jelly", "red", "bean",
                "whipped", "cream", "ice", "more", "less", "no", "sugar", 'and',
                "normal", "large", "small", "medium"]
    commands = ["want", "desire", "special", "order", "recommend", "price", 'and']
    all_important_words = keywords + commands

    def __init__(self):
        self.order_commands = {
            'special_order':[r'.*order.*special.*'],
            r'price_inquiry':[r'.*much.*that', r'.*much.*this'],
            r'suggested_order':[r'.*want.*that', r'.*sounds\sgood.*',
                                r'.*get.*that.*', r'.*get.*special.*', 
                                r'.*order.*special.*', r'order.*that.*'],
            r'describe_specials': [r'.*special.*', r'.*\recommend.*'],  
            r'menu_inquiry':[r'.*menu.*', r'.*choice.*', r'.*option.*'],
            r'single_order': [r'.*want.*', r'.*order.*',r'.*get.*'],
            r'checkout': [r'.*checkout.*', r'.*done.*', r'.*bill.*',
                          r'all.*', r'.*no.*']}
    
    # Cleans up strings by removing unncessary words, autocorrects 
    # Str -> Str
    def reply_cleaner(self, text):
        text_lowered = text.lower()
        tokenizer = RegexpTokenizer(r'\w+')
        tokenized_text = tokenizer.tokenize(text)
        for word in tokenized_text:
            for correctly_spelt_word in self.all_important_words:
                if word == 'no': continue # too easy to autocorrect incorrectly
                elif edit_distance(word, correctly_spelt_word) == 1:
                    word_index = tokenized_text.index(word)
                    tokenized_text[word_index] = correctly_spelt_word
        cleaned_string = " ".join(tokenized_text)         
        # print(f"I'm interpreting this as {cleaned_string}")
        return cleaned_string
    
    # Str -> listof Tokens or Str
    def essential_words(self, reply, form = "list"):
        #!print(f"{reply} is being stripped to its essentials")
        tokenized_reply = word_tokenize(reply)
        essential_words = []
        for word in tokenized_reply:
            if word in self.keywords:
                essential_words.append(word)        
        if form == "string":
            return " ".join(essential_words)  
        else: # form == "list":
            return essential_words
               
                       
    def exit(self, reply):
        #!print("checking for exit statements")
        for command in self.exit_commands:
            if command in reply:
                reply = input("Have a nice day!")
                return True

        
    def greet(self):
        self.name = input("""
        Hi! I'm Hamilton. I work for Abi's Bubble Tea House!
        I'll help you with your order today. You can talk to me to place 
        your order, or ask about today's special! What's your name? 
        """)
        self.chat()
        
    def chat(self):
        #!print("I am back in the chat method")
        request = self.reply_cleaner((input(f"What can I do for you {self.name}?\n")))
        while self.exit(request) != True:
            #self.exit(request)
            #!print(f"request is {request}1")
            request = self.reply_cleaner(input(self.match_reply(request)))
            #!print(f"request is {request}2")
            #!print("I am back in the chat method2")
            
        
    def match_reply(self, reply):
        #!print(f"reply is {reply}")
        for intent, phrasing in self.order_commands.items():
            for regex_pattern in phrasing:
                #!print(f"searching {regex_pattern} in {intent}")
                found_match = re.match(regex_pattern, reply)
                if found_match:
                    if intent == 'describe_specials':
                        produced = self.describe_special_intent()
                        #!print(f"{produced} is in match_reply")
                        return produced
                    elif intent == 'menu_inquiry':
                        menu_printer("drinks")
                        menu_printer("toppings")
                        print("There you go! I've pointed out the special too!")
                        return
                    elif intent == 'suggested_order' or intent == 'single_order':
                        if intent == 'suggested_order':
                            suggested = "order" + self.last_reference + reply
                            self.single_order_intent(suggested)
                        else:
                            self.single_order_intent(reply)
                        print("I am printing out the drinks you've ordered")
                        for drink in self.order_items:
                            drink.print_drink()
                        print("Would you like to add anything else "
                                      "or go to the checkout?")
                        return
                    elif intent == 'price_inquiry':
                        return self.price_inquiry_intent()

                    elif intent == 'checkout' and self.order_items != []:
                        return self.checkout_intent()
        # occurs if nothing has been found            
        return self.no_match_intent()
    
    # produces True or False, depending on if a new drink should be produced.
    # A new drink is signified by overlapping descriptors that do not occur in 
    # "toppings".
    def new_drink(self, descriptor, currentDrink, drink_lastword):  
        
        if descriptor in drink_strings:
            full_drink_name = None
            for key in drinks_menu.keys():
                if descriptor in key:
                    full_drink_name = key
                    break
        # if drinks have different names, then they are different
        if full_drink_name != currentDrink.name : 
            return True
        
        elif (descriptor in ["large", "medium", "small"] 
              and currentDrink.size != None):
            return True
    
    # produces a list of Drink(s) based on given words
    def words_to_drink(self, important_words):
        #!print("words_to_drink is running")
        name, toppings, size, sugar, ice = None, ["nothing"], None, -1, -1
        produced_drinks, drink_lastword, toppings_lastword = [], None, None
        last_word_and = None
        adjective = None
        drinks_started = 0
        for descriptor in important_words:
            #!print(f"Currently analyzing word: {descriptor}")
            drink_so_far = Drink(name, toppings, size, sugar, ice)
            if descriptor in drink_strings:
                # this signifies the end of the current drink, and the start
                # of a new drink. 
                if drinks_started !=0 and self.new_drink(descriptor, drink_so_far, drink_lastword) and last_word_and == False:
                    #!print("This is the end of the current drink, starting new")
                    produced_drinks.append(drink_so_far)
                    drink_lastword = True
                    name, toppings, size, sugar, ice = None, ["nothing"], None, -1, -1
                    drinks_started += 1
                    
                if drinks_started == 0: drinks_started += 1 
                # finding the "full name" of the descriptor
                for key in drinks_menu.keys():
                    #!print("This is running")
                    if descriptor in key:
                        name = key
                        break
                #!print(f"The name is {name}")
                drink_lastword = True
                last_word_and = False
                
            elif descriptor in toppings_strings:
                last_word_and = False
                #!print(f"{descriptor} is a topping")
                current_topping = None
                drink_lastword = False
                # finding full name of toppings:
                for key in toppings_menu.keys():
                    if descriptor in key:
                        current_topping = key
                        break
                if toppings == ["nothing"]:
                    toppings[0] = current_topping  
                else:
                    if current_topping != toppings_lastword:
                        toppings.append(current_topping)
                toppings_lastword = current_topping
                    
            elif descriptor in ["less", "normal", "more"]:
                last_word_and = False
                #!print(f"{descriptor} is a adj")
                drink_lastword = False
                adjective = descriptor
                
            elif descriptor == 'ice':
                last_word_and = False
                #!print(f"{descriptor} is a ice")
                drink_lastword = False
                ice = adjective
                adjective = None
                
            elif descriptor == 'sugar':
                last_word_and = False
                #!print(f"{descriptor} is a sugar")
                drink_lastword = False
                sugar = adjective
                adjective = None
                
            elif descriptor in ["large", "medium", "small"]:
                if last_word_and == True:
                    #!print("This is the end of the current drink, starting new")
                    produced_drinks.append(drink_so_far)
                    drink_lastword = True
                    name, toppings, size, sugar, ice = None, ["nothing"], None, -1, -1
                    drinks_started += 1                    
                #!print(f"{descriptor} is a size")
                size = descriptor
                drink_lastword = False
                
                
            elif descriptor == "and":
                last_word_and = True
                        
        #!print(f"{drinks_started} drinks started")
        #!print(f"{len(produced_drinks)} is how many drinks produced")        
        if drinks_started != len(produced_drinks):
            #!print("I am producing extra drinks")
            #!print(name, toppings, size, sugar, ice)
            finished_drink = Drink(name, toppings, size, sugar, ice)
            produced_drinks.append(finished_drink) 
            #!print("I'm done producing extra drinks.")
            #!print(f"{drinks_started} drinks started")
            #!print(f"{len(produced_drinks)} is how many drinks produced")             
         
        for bubble_tea in produced_drinks:
            self.order_items.append(bubble_tea)
            bubble_tea.print_drink()
            
        return produced_drinks
    

    def single_order_intent(self, reply):
        #!print("this is a single_order_intent")
        important_words = self.essential_words(reply, "list")
        requested_drinks = self.words_to_drink(important_words)
        #!print("finished_drinks have been receive, returning..")
        return self.finish_off_drinks(requested_drinks)
        #for word in important_words:
        
    # loops until a valid response is received from the user
    # drink_part can be anyof "size", "ice", "sugar", "toppings"
    # returns a string, corresponding to a valid menu item (in exact wording)
    def finish_off_drink_mechanic(self, drink_part, drink):   
        correct_answer = False
        repeated = False
        
        if drink_part == "toppings":
            toppings = ["nothing"]
            want = input("Do you want any toppings? ")
            if "no" in self.reply_cleaner(want): return ["nothing"]
            while correct_answer == False:
                if repeated == True:
                    print("""Sorry, that's not a response I was looking for.
                    If you're unsure, feel free to ask for a menu!\n""")
                response = self.essential_words(self.reply_cleaner(input (f"""
                What toppings would you want in {drink.name}?\n""")))
                if any(x in response for x in self.menu_words):
                    menu_printer("toppings")
                    response = self.essential_words(self.reply_cleaner(input("\n")))
                    
                current_topping = None
                drink_lastword = False
                toppings_lastword = None
                # finding full name of toppings:
                for word in response:
                    for key in toppings_menu.keys():
                        if word in key:
                            current_topping = key
                            correct_answer = True
                            break   
                    if toppings == ["nothing"]:
                        toppings[0] = current_topping  
                    else:
                        if current_topping != toppings_lastword:
                            toppings.append(current_topping)
                    toppings_lastword = current_topping
                
                repeated = True
            return toppings
        
        if drink_part == "ice" or drink_part == "sugar" or drink_part == "size":
                while correct_answer == False:
                    if repeated == True:
                        print("Sorry, I didn't quite get that.\n")
                    if drink_part == "size":
                        response = self.reply_cleaner(input (f"""
                        What size would you want {drink.name} to be? You can 
                        pick between small, medium, or large.\n"""))
                        correct_answer = next((x for x in ['small', 'medium', 'large'] 
                                               if x in response ), False)  
                    else:
                        response = self.reply_cleaner(input (f"""
                        How much {drink_part} would you want {drink.name} to have? 
                        You can pick between less, normal, or more {drink_part}.\n"""))
                        correct_answer = next((x for x in ['less', 'normal', 'more'] 
                                           if x in response ), False)
                    repeated = True
                return correct_answer

    def finish_off_drinks(self, listofDrinks):
        for drink in listofDrinks:
            if drink.size == None:
                drink.size = self.finish_off_drink_mechanic("size", drink)
                print(f"Gotcha, a {drink.size} {drink.name} ")
                
            if drink.toppings == ["nothing"]:
                drink.toppings = self.finish_off_drink_mechanic("toppings", drink)
                toppings_as_string = ", ".join(drink.toppings)
                print(f"A {drink.size} {drink.name} with {toppings_as_string} "
                      "as toppings, great!")
            
            if drink.ice == -1:
                drink.ice = self.finish_off_drink_mechanic("ice", drink)
                print(f"Fine choice {self.name}! ")
            
            if drink.sugar == -1:
                drink.sugar = self.finish_off_drink_mechanic("ice", drink)
                print("Delicious! ")
        #!print("returning listofDrinks")        
        return listofDrinks
 
    def describe_special_intent(self):
        print(f"Today's special is the {todays_special}! It is $1.5 "
                      "off its original price!\n")
        self.last_reference = todays_special
        #return self.reply_cleaner(reply)

    def price_inquiry_intent(self):
        #!print("this is a price_inquiry_intent")
        reference = self.last_reference
        reply = input(f"The {reference} is ${big_menu[reference]}. \n")
        return self.reply_cleaner(reply)
    
    
    
    def no_match_intent(self):
        self.unable_to_communicate += 1
        if (self.unable_to_communicate == 3):
            return ("""
            We seem to have trouble communicating! Could you give us a call at
            (647)-542-9669 to continue with your order? Thanks!
            """)
        else:
            responses = ("Could you phrase that in a different way? ",
                        "Sorry, I didn't understand, could you say it differently ",
                        "Can you type that out again? I don't understand. ")
            return random.choice(responses)
        
    def checkout_intent(self):
        price = 0
        for drink in self.order_items:
            price += drink.total_cost()
        total_cost = round(price * 1.13, 2)
        postal_code_reply = (input(f"""
        Your total is ${round(price, 2)}. We'll deliver it to your door,
        at a cost of $2. Plus tax, the total is {total_cost+2}.
        You can pay at the door. Input your postal code:\n""")).lower()
        #!print(postal_code_reply)
        postal_code = re.findall(r'[a-z]\d[a-z]', postal_code_reply)[0]
        print(f"Confirmed! Your drink will take around {distance(postal_code)} "
              "minutes to get there from our Toronto Yonge Street location! "
              f"Thanks {self.name} for ordering with us!")
        self.comments()
        print(f"Bye {self.name}, I hope to see you again!")
        quit()
              
    def comments(self):
        reply = input("Do you want to give me feedback on how I did? ")
        analyzer = SentimentIntensityAnalyzer()
        score = analyzer.polarity_scores(reply)
        if score['neg'] == 0:
            feedback = input("Great! Go ahead!\n")
            feedback_score = analyzer.polarity_scores(feedback)
            if feedback_score['pos'] >= 0:
                print("That's so nice! Thanks for your feedback!")
            else:
                print("We're always trying to improve Hamilton, thanks for your "
                      "invaluable feedback that will make Hamilton better!") 
        else:
            print("Alright, thanks again for your order!")
        return
    
bot = BTBot()
bot.greet()
