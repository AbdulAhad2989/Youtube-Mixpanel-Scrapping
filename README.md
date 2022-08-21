**Scraping and Analyzing Influencer Marketing Data from Youtube** 
**Scraping Soft Metrics from Youtube**
- This project involved Scarping YouTube video data to gain insights into the soft metrics of the videos. 
- Secondly this project involved scarping hard metrics data from Mixpanel using its JQL API.
- Data was dumped on to the BigQuery Data Warehouse. 
- The hard metrics and soft metrics were linked through referee id provided in the posts.
- Then reporting was done on Google Sheets to show the hard and soft metrics of different videos together. 
- Different ratios were used to gauge the impact of the videos and ROI generated.

First of all, we look at YouTube hard metrics data scraping:
-The scraping was done through YouTube Data API for videos. Link for the API is present below.
- https://developers.google.com/youtube/v3/docs/videos/list
- Language used for this purpose was Python. 
- Google Sheet’s Apps Script(Java Script) was used to passed through specific video ids to the Python Scraping Script.
- Below is the link to the Google Sheet. 
- https://docs.google.com/spreadsheets/d/1IzVvOWg7Yd5bJVxnbrVtJzMP0rxOaS65slXm1ITFn44/edit#gid=638558079
- Below is the Apps Script used to pass the columns provided in the Google Sheet as variable to Scraping Script.

```


 	function Read_Soft_Data(){
	var sheet = SpreadsheetApp.getActive().getSheetByName("Inputsheet");
	var range = sheet.getRange(1,1, sheet.getLastRow(),sheet.getLastColumn());
	var url = range.getValues();  
		url.forEach(function(row, index) {
  				if (index !== 0) { 
    				var sheetData = {
	     				"Influencer Name ":row[0],
	     				"post_url" :  row[1],
	     				"platform" :  row[2],
	     				"influencer_link" :  row[3],
	     				"influencer_id" :  row[4],
	     				"post_date" :  row[5],
	     				"spent" :  row[6],
	     				"category" :  row[7],
	     				"sub_cateogry" :  row[8],
	     				"content_type" :  row[9],
	     				"pull_data" :  row[10],
	     				"trigger_data_soft_metric" :  row[11],
	     				"hard_metric" :  row[12]          
    						}    
      				var dt1 = new Date(); // today's date
      				var   dt2 =new Date(sheetData["post_date"]);      
      				var t1 = dt1.getTime(),
          				t2 = dt2.getTime();      
      				var diffInDays = Math.floor((t1-t2)/(24*3600*1000));
      				if(diffInDays<=124 && sheetData['platform']=='Youtube'){
        				var data = JSON.stringify(sheetData);
        				Logger.log(data)
        				pubsub('savyour-prod','topic_infl_soft_metric',{'test':'test'},data);
      					}
      					else if(diffInDays>124){        
      					}       
  				}
			})}

```

- The Post URL from this Apps Script is passed to the Google Cloud Function which runs the Python Script for YouTube Data API. This API in turn scraps the video for the soft metrics of the video.
- This script also dumps the hard functions values into BigQuery.
- The python script for this scrap procedure is given below.

```

import base64
import json
import pandas_gbq
from google.cloud import bigquery
from googleapiclient.discovery import build
import pandas as pubsub_message_jsonpd
import pandas_gbq
from pytube import extract
from datetime import datetime,timedelta
api_key='AIzaSyBAAXzs1K0nvivr7DK39Z8YiAnWgXx3N4w'

def hello_pubsub(event, request):
#     """Triggered from a message on a Cloud Pub/Sub topic.
#     Args:
#          event (dict): Event payload.
#          context (google.cloud.functions.Context): Metadata for the event.
#     """
    pubsub_message = base64.b64decode(event['data']).decode('utf-8')
    # print(pubsub_message)
    pubsub_message_json = json.loads(pubsub_message)
    # print(pubsub_message_json)

    
    try:
      referraluserid=pubsub_message_json['influencer_id']
    except:
      referraluserid=None
    try:
      post_url=pubsub_message_json['post_url']
    except:
      post_url=None
    try:
      platform=pubsub_message_json['platform']
    except:
      platform=None
    try:    
      influencer_link=pubsub_message_json['influencer_link']
    except:
      influencer_link=None
    try:    
      influencer_id=pubsub_message_json['influencer_id']
    except:
      influencer_id=None
    try:    
      post_date=pubsub_message_json['post_date']
    except:
      post_date=None
    try:    
      spent=pubsub_message_json['spent']
    except:
      spent=None
    try:    
      category=pubsub_message_json['category']
    except:
      category=None
    try:  
      sub_cateogry=pubsub_message_json['sub_cateogry']
    except:
      sub_cateogry=None
    try:  
      content_type=pubsub_message_json['content_type']
    except:
      content_type=None  
    print(post_url)

    youtube=build('youtube', 'v3', developerKey=api_key)
    request=youtube.videos().list(
        part="snippet,statistics",
        id=extract.video_id(post_url)
        

    )

    video_id=extract.video_id(post_url)
    response=request.execute()
    print(response)

    items_in_video=response.get('items')
    try:
        view_count=items_in_video[0]['statistics']['viewCount']
    except:
        view_count=None
    try:    
        like_count=items_in_video[0]['statistics']['likeCount']
    except:
        like_count=None
    try:
        comment_count=items_in_video[0]['statistics']['commentCount']
    except:
        comment_count=None
    # id=a[0]['id']
    try:
        channel_id=items_in_video[0]['snippet']['channelId']
    except:
        channel_id=None
    try:
        title=items_in_video[0]['snippet']['title']
    except:
        title=None
    try:  
        channel_title=items_in_video[0]['snippet']['channelTitle']
    except:
        title=None
    try:        
        published_date=items_in_video[0]['snippet']['publishedAt']
    except:
        published_date=None
    try:       
        url=post_url
    except:
        url=None    

    Channel_id_from_channel_api=items_in_video[0]['snippet']['channelId']
    Channel_id_from_channel_api
    channel_data=youtube.channels().list(
            part="snippet,statistics",
            id=Channel_id_from_channel_api
    )
    channel_data_execute=channel_data.execute()
    channel_data_items=channel_data_execute.get('items')
    try:
        channel_data_items=channel_data_execute.get('items')
        subscriber_count=channel_data_items[0]['statistics']['subscriberCount']
    except:
        subscriber_count=None
        print(subscriber_count)
    Channel_Title=channel_data_items[0]['snippet']['title']

    new_dict= dict([('video_id', video_id),
                ('post_url', url),
                ('video_title',title),
                ('published_date',published_date),
                ('channel_id',channel_id),
                ('channel_title',channel_title),
                ('subscriber_count',subscriber_count),
                ('view_count',view_count),
                ('like_count', like_count),
                ('comment_count', comment_count)
               ])   
 
    execution_time = datetime.now()+ timedelta(hours=5)
    d1 = datetime.strptime(published_date,"%Y-%m-%dT%H:%M:%SZ")
    new_format = "%Y-%m-%d %H:%M:%S"
    d1.strftime(new_format)
    post_date_calculated=d1+ timedelta(hours=5)
    post_date_calculated

    PROJECT_ID = "xxx"
    DATASET_ID = "xxx"
    TABLE_ID = "xxx"

    client = bigquery.Client(project=PROJECT_ID)
    table_id = "[PROJECT_ID].[DATASET_ID].[TABLE_ID]"

    Row_list =[] 
    new_list=[post_url,platform,influencer_link,referraluserid,post_date,spent,category,sub_cateogry,content_type,video_id,url, title,post_date_calculated,channel_id,channel_title,subscriber_count,view_count,like_count,comment_count,execution_time]
    Row_list.append(new_list) 

    client = bigquery.Client(project=PROJECT_ID)
    table_id = "xxx.xxx"
    table = client.get_table(table_id)
    errors = client.insert_rows(table, Row_list)  # Make an API request.
    if errors == []:
        print("New rows have been added.")
    # )
    # url=request_json['post_url']
    # print(url)

    # print()
    # return "success"


```
