---
name: bilibili-miniapp-mobile-catalog
description: |
  Locate Bilibili mobile mini-game entries when PC game-center search has no exact
  title or an ad/developer video does not expose the playable link. Cross-check
  public mobile catalog, miniapp identity and game detail metadata.
---

# Bilibili mobile mini-game lookup

Use for a Bilibili mini-game source-discovery gap. A PC catalog miss does not
establish absence from the mobile mini-game catalog. A same-name video alone
does not establish game identity or the advertised level.

1. Inspect the official mobile page `https://app.biligame.com/` and its current
   JavaScript for search paths and device parameters. In the verified iOS route,
   `DEVICE_TYPE` maps iOS to `sdk_type=2`. Endpoint details may change; re-check
   the official client if a request fails rather than guessing authentication.
2. The verified public keyword route was:

   ```sh
   title='目标小游戏名称'
   curl --get 'https://line3-h5-mobile-api.biligame.com/game/center/h5/search/page' \
     --data-urlencode "keyword=$title" --data 'page_num=1' --data 'page_size=20' \
     --data 'sdk_type=2' --data 'client=h5'
   ```

   Require `code=0` before interpreting `data.list`. An exact mini-game result
   had `source=4`, `game_base_id`, `small_game_link` and
   `video_info.bv_id`. Inspect pagination/ambiguity rather than treating the
   first page as exhaustive. The similarly named `/search/smallgame` route was
   a recommendation widget and returned a login error; it was not the verified
   keyword route.
3. Extract the app ID from the returned official
   `https://miniapp.bilibili.com/game/<appId>` URL. Cross-check:
   - `https://miniapp.bilibili.com/api/miniapp/info/main?appId=<appId>`:
     `appId`, `name`, `companyName`.
   - `https://line3-h5-mobile-api.biligame.com/game/center/h5/detail/gameinfo/v2?game_base_id=<id>&sdk_type=2&client=h5`:
     `game_base_id`, `small_game_link`, `developer_name`, `bv_id`.

   Compare title, developer and any already-observed developer video; retain
   ambiguity instead of choosing an approximate title. Keep raw responses in
   the project's ignored private evidence directory.
4. The web game entry can be only an “open in Bilibili” landing page. Separate
   metadata identity, successful native launch, and actual gameplay observation.
   Launch through the authorized app surface when needed; do not invent a
   standalone playable URL or copy private account cookies to bypass the shell.

Verified on 2026-10-20: PC search had no exact title; the public mobile route
returned an exact result linked to the known developer video, both metadata
checks agreed, and native launch reached a banana tutorial. This proved the
entry, not the gray-board ad scene. No credentials were required for these
metadata calls. Do not infer advertised-level availability, rules or audio
parity from identity resolution, or keep searching indefinitely after the user
has chosen independent evolution.
