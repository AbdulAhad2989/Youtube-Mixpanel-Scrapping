Scraping and Analyzing Influencer Marketing Data from Youtube and Mixpanel

Scraping Soft Metrics from Youtube
> This project involved Scarping YouTube video data to gain insights into the soft metrics of the videos. 
> Secondly this project involved scarping hard metrics data from Mixpanel using its JQL API.
> Data was dumped on to the BigQuery Data Warehouse. 
> The hard metrics and soft metrics were linked through referee id provided in the posts.
> Then reporting was done on Google Sheets to show the hard and soft metrics of different videos together. 
> Different ratios were used to gauge the impact of the videos and ROI generated.

First of all, we look at YouTube hard metrics data scraping:
> The scraping was done through YouTube Data API for videos. Link for the API is present below.
https://developers.google.com/youtube/v3/docs/videos/list
> Language used for this purpose was Python. 
> Google Sheet’s Apps Script(Java Script) was used to passed through specific video ids to the Python Scraping Script.
> Below is the link to the Google Sheet. 
https://docs.google.com/spreadsheets/d/1IzVvOWg7Yd5bJVxnbrVtJzMP0rxOaS65slXm1ITFn44/edit#gid=638558079
> Below is the Apps Script used to pass the columns provided in the Google Sheet as variable to Scraping Script.
