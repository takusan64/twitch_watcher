# twitch_watcher
好きな配信者のTwitch配信を検知し、自動で開くアプリ

## 内部動作フロー
1. 「client_id」と「client_secret」からアクセストークンを取得する
  参考：[Twitch APIに必要なOAuth認証のアクセストークンを取得しよう](https://qiita.com/pasta04/items/2ff86692d20891b65905)
1. 配信者チャンネルページのURLからuserIdを取得する
  参考：[Twitch API Reference Get Users](https://dev.twitch.tv/docs/api/reference#get-users)
  ```json
  // 例：配信者のチャンネルURLが 「https://www.twitch.tv/twitchdev」 の場合、末尾の「twitchdev」を利用する
  // request(get) : https://api.twitch.tv/helix/users?login=twitchdev
    // header: {Client-ID:{client_Id}, Authorization:Bearer {access_token}}

  // 存在するチャンネルの場合
  {
    "data": [
      {
        "id": "141981764", // ここが「userId」
        "login": "twitchdev",
        "display_name": "TwitchDev",
        "type": "",
        "broadcaster_type": "partner",
        "description": "Supporting third-party developers building Twitch integrations from chatbots to game integrations.",
        "profile_image_url": "https://static-cdn.jtvnw.net/jtv_user_pictures/8a6381c7-d0c0-4576-b179-38bd5ce1d6af-profile_image-300x300.png",
        "offline_image_url": "https://static-cdn.jtvnw.net/jtv_user_pictures/3f13ab61-ec78-4fe6-8481-8682cb3b0ac2-channel_offline_image-1920x1080.png",
        "view_count": 5980557,
        "email": "not-real@email.com",
        "created_at": "2016-12-14T20:32:28Z"
      }
    ]
  }

  //存在しないチャンネルの場合
  {
    "data": []
  }
  ```
  (Twitch配信のURLの末尾pathを利用する)
1. userIdから配信しているか取得する
  参考：[Twitch API Reference Get Streams](https://dev.twitch.tv/docs/api/reference#get-streams)
  ```json
  // 例：配信者の「user_Id」が 「141981764」 の場合
  // request(get) : https://api.twitch.tv/helix/streams?user_id=141981764
    // header: {Client-ID:{client_Id}, Authorization:Bearer {access_token}}

  // 配信中の場合
  {
    "data": [
      {
        "id": "141981764324",
        "user_id": "141981764",
        "user_login": "twitchdev",
        "user_name": "TwitchDev",
        "game_id": "494131",
        "game_name": "Little Nightmares",
        "type": "live",
        "title": "hablamos y le damos a Little Nightmares 1",
        "viewer_count": 78365,
        "started_at": "2021-03-10T15:04:21Z",
        "language": "es",
        "thumbnail_url": "https://static-cdn.jtvnw.net/previews-ttv/live_user_auronplay-{width}x{height}.jpg",
        "tag_ids": [
          "d4bb9c58-2141-4881-bcdc-3fe0505457d1"
        ],
        "is_mature": false
      },
      ...
    ],
    "pagination": {
      "cursor": "eyJiIjp7IkN1cnNvciI6ImV5SnpJam8zT0RNMk5TNDBORFF4TlRjMU1UY3hOU3dpWkNJNlptRnNjMlVzSW5RaU9uUnlkV1Y5In0sImEiOnsiQ3Vyc29yIjoiZXlKeklqb3hOVGs0TkM0MU56RXhNekExTVRZNU1ESXNJbVFpT21aaGJITmxMQ0owSWpwMGNuVmxmUT09In19"
    }
  }

  //配信していない場合
  {
    "data": [],
    "pagination": {}
  }
  ```

[**Tips**]
- アクセストークンは60日までしか有効でないため、[Twitch API Reference Validating Requests](https://dev.twitch.tv/docs/authentication#validating-requests)を使って検証可能
(アクセストークンの再取得でトークンの更新を行う)