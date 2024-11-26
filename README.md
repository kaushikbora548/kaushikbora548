from googleapiclient.discovery import build
import pandas as pd

# Set up the YouTube API client
api_key = 'YOUR_API_KEY'  # Replace with your API key
youtube = build('youtube', 'v3', developerKey=api_key)

# Channel ID (from the URL of the channel)
channel_id = 'YOUR_CHANNEL_ID'  # Replace with the channel ID

# Function to get videos
def get_video_links(channel_id):
    video_links = []
    # Make a request to the API for the channel's uploads
    request = youtube.channels().list(part="contentDetails", id=channel_id)
    response = request.execute()

    # Extract the playlist ID of the channel's uploads
    playlist_id = response['items'][0]['contentDetails']['relatedPlaylists']['uploads']

    # Get all videos in the playlist
    request = youtube.playlistItems().list(part="snippet", playlistId=playlist_id, maxResults=50)
    while request:
        response = request.execute()
        for item in response['items']:
            video_links.append(f"https://www.youtube.com/watch?v={item['snippet']['resourceId']['videoId']}")
        request = youtube.playlistItems().list_next(request, response)
    
    return video_links

# Get video links and save them to an Excel file
video_links = get_video_links(channel_id)
df = pd.DataFrame(video_links, columns=["Video Links"])
df.to_excel("video_links.xlsx", index=False)
