#!/usr/bin/python3

import requests
import webbrowser
import pickle
import sys
import configparser
import argparse
import json
from datetime import datetime, timedelta
import feedparser
import time
import os

#Get Pickle data
def __init__():
	redirect_uri = 'https://github.com/Dave-Vallance/Auto-Pocket.git'
	request_url = 'https://getpocket.com/v3/oauth/request'
	auth_url = 'https://getpocket.com/auth/authorize?request_token='
	access_token_url = 'https://getpocket.com/v3/oauth/authorize'
	if os.path.exists(home + '/.config/Auto-Pocket/data.pickle'):
		pickle_file = home + '/.config/Auto-Pocket/data.pickle' 
	else:
		pickle_file = 'data.pickle'
	
	with open(pickle_file, 'rb') as f:
		pickle_data = pickle.load(f)
	if 'request token' not in pickle_data:
		print('First time authorization required...')
		#Start authorization
		token_req = requests.post(
			request_url,
			data = {'consumer_key': pickle_data['consumer key'],
			'redirect_uri':redirect_uri}
			)

		status = token_req.status_code #if 200, it means all ok.
		if int(status) != 200:
			print(token_req.headers['X-Error'])#gives error reason
			print("Terminating")
			sys.exit(1)
		pickle_data['request token'] = token_req.text.strip('code=')

		#Have the user authenticate the app
		full_url = auth_url + pickle_data['request token'] + '&redirect_uri=' + redirect_uri
		webbrowser.open_new_tab(full_url)
		input("Please authorize the application then press Enter to continue...")

		#Use the token + the consumer key to request an access token
		access_req = requests.post(access_token_url,
       				data = {'consumer_key': pickle_data['consumer key'],
				'code': pickle_data['request token']}
				)
		status = access_req.status_code
		if int(status) != 200:
			print(access_req.headers['X-Error'])#gives error reason
			print("Terminating")
			sys.exit(1)
		pickle_data['access token'] = access_req.text.strip('access_token=').split('&')[0]

		#save the tokens
		with open(pickle_file, 'wb') as f:
			# Pickle the 'data' dictionary using the highest protocol available.
			pickle.dump(pickle_data, f, pickle.HIGHEST_PROTOCOL)

		return pickle_data['consumer key'], pickle_data['request token'], pickle_data['access token']

	else:
		return pickle_data['consumer key'], pickle_data['request token'], pickle_data['access token']



def cli_add_article(url, consumer_key, access_key, tag):
	'''
	For adding a single article from command line
	'''
	pocket_add = requests.post('https://getpocket.com/v3/add',
				data= {'url': url,
		'consumer_key': consumer_key,
		'access_token': access_key,
		'tags': tag})
				
	status = pocket_add.status_code
	#TODO If status code != 200, then....

	return pocket_add.status_code
	

def add_articles(url_list, consumer_key, access_key, tags=None):
	
	
	payload = {
		'actions': [],
		'consumer_key': consumer_key,
		'access_token': access_key
		}

	for url in url_list:
		payload['actions'].append({
					'action' : 'add',
					'url' : url,
					'tags': tags
					})

	pocket_add = requests.post('https://getpocket.com/v3/send',json=payload)
				
	status = pocket_add.status_code
	#TODO If status code != 200, then....

	return pocket_add.status_code
	


def ret_all_articles(consumer_key, access_key, tag, detail_type='complete'):
	
	pocket_ret = requests.post('https://getpocket.com/v3/get',
		data= {'consumer_key':consumer_key,
		'access_token': access_key,
		'tag':tag})

	#TODO If status code != 200, then....

	json_data = pocket_ret.json()
	print(pocket_ret.status_code)
		
	return pocket_ret.status_code, json_data


def delete_article(consumer_key, access_key, item_list):

	payload = {
		'actions': [],
		'consumer_key': consumer_key,
		'access_token': access_key
		}
	
	for item_id in item_list:
		payload['actions'].append({
					'action' : 'delete',
					'item_id' : int(item_id)
					})

	pocket_del = requests.post('https://getpocket.com/v3/send',json=payload)
				
	status = pocket_del.status_code

	return status


def delete_article_check(ret_data, delete_after):
	article_id_list = []
	for article in ret_data['list']:	
		time_added = datetime.fromtimestamp(int(ret_data['list'][article]['time_added']))
		archived = int(ret_data['list'][article]['status'])
		favorite = int(ret_data['list'][article]['favorite'])
		article_expiry = timedelta(days=int(delete_after))
		if (current_time - article_expiry) > time_added:
			print('Over %s day(s) old...' % delete_after)
			if archived == 1:
				print('But is an archived article, keeping...')
			elif favorite == 1:
				print('But is a favorite article, keeping...')
			else:
				print('deleting...')				
				item_id = article
				article_id_list.append(item_id)
		else:
			print('under %s day(s) old, keeping...' % delete_after)

	if not article_id_list:
		print('Nothing to delete... moving on.')
	else:
		status = delete_article(consumer_key, access_key, article_id_list)
		print(status)


##########SCRIPT###########

#Argparsing
parser = argparse.ArgumentParser(description="Auto Pocket, automatically "
	"adds articles to pocket from given feeds in the config file")
parser.add_argument("-u", "--url", help="Allows uploading a single article from the command line. Usefull for integration with other apps e.g newsbeuter")
args = parser.parse_args()

#get users home dir
home = os.path.expanduser("~")
print(home)


'''
Here I want to have the option to keep my configs seperate from the repository. 
'''
if os.path.exists(home + '/.config/Auto-Pocket/Auto-Pocket.config'):
	config_file = home + '/.config/Auto-Pocket/Auto-Pocket.config'
else:
	config_file = 'Auto-Pocket.config'

#Configparsing
config = configparser.ConfigParser()
config.read(config_file)

#Section parsing
feed_list = config.sections()
general_settings = feed_list.pop(0)


#Get Current time
current_time = datetime.utcnow()
print(current_time)

#get general configs
auto_pocket_tag = config.get(general_settings, 'Auto-Pocket-Tag')
cli_tag = config.get(general_settings, 'cli_tag')
cli_delete_after = config.get(general_settings, 'cli_delete_after')



#Get keys
consumer_key, request_key, access_key = __init__()

if args.url is not None:
	tags = auto_pocket_tag + ', ' + cli_tag
	status_code = cli_add_article(args.url, consumer_key, access_key, tags)
	print(status_code)

else:
	status_code, ret_data = ret_all_articles(consumer_key, access_key, cli_tag)
	delete_article_check(ret_data, cli_delete_after)	

	for feed in feed_list:

		#Retrieving Configs
		feed_tag = config.get(feed, 'tag')
		delete_after = config.get(feed, 'delete_after')
		#Adding Configs	
		url = config.get(feed, 'url')
		days_to_add = config.get(feed, 'days_to_add')

		####RETRIEVE POCKET LIST AND DELETE OLD ARTICLES #####

		status_code, ret_data = ret_all_articles(consumer_key, access_key, feed_tag)

		#print(json.dumps(ret_data, sort_keys=True, indent=4))
		delete_article_check(ret_data, delete_after)

		###########ADD NEW ARTICLES FROM FEED ############

		article_range = timedelta(days=int(days_to_add))
		rss_data = feedparser.parse(url)
		tags = auto_pocket_tag + ', ' + config.get(feed, 'tag')
		url_list = []
		for article in rss_data.entries:
			published = datetime.fromtimestamp(time.mktime(article.published_parsed))
			if (current_time - article_range) > published:
				print('Too old, skipping')
				print(article.title)
				print(published)
			else:
				print('going to add this!')
				print(published)
				print(article.title)
				link = article.link
				url_list.append(link)
		status = add_articles(url_list, consumer_key, access_key, tags)
		print('status = ' + str(status))
				


