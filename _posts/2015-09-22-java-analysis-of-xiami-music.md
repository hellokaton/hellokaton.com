---
layout: post
title: java解析虾米音乐
categories: java
tags: 爬虫
---

下面的java代码可以解析虾米音乐。

<!-- more -->

```java
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.util.HashMap;
import java.util.Map;

import blade.kit.http.HttpRequest;
import blade.kit.json.JSONArray;
import blade.kit.json.JSONObject;

public class XiaMi {
	
	private static final String UA = "Mozilla/5.0 (iPhone; CPU iPhone OS 7_0 like Mac OS X; en-us) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11A465 Safari/9537.53";
	
	public static Map<String, Object> getXiaMiData(String id) {
		Map<String, Object> map = new HashMap<String, Object>();
		
		String url = "http://www.xiami.com/song/playlist/id/"+ id +"/object_id/0/cat/json";
		
		String content = HttpRequest.get(url)
				.userAgent(UA)
				.referer("http://www.xiami.com/song/playlist/id/" + id).body();
		
		try {
			
			JSONObject jsonObject = new JSONObject(content);
			if(null == jsonObject || !jsonObject.getBoolean("status")){
				return map;
			}
			
			JSONObject data = jsonObject.getJSONObject("data");
			
			if(null == data){
				return map;
			}
			
			JSONArray trackList = data.getJSONArray("trackList");
			if(null == trackList){
				return map;
			}
			
			JSONObject track = trackList.getJSONObject(0);
			
			String songurl = track.getString("location");
			songurl = xiamidecode(songurl);
			
			String songpic = track.getString("album_pic").replace(".jpg", "_2.jpg");
			String title = track.getString("title");
			String singer = track.getString("artist");
			String lyricurl = track.getString("lyric");
			
			map.put("id", id);
			map.put("songurl", songurl);
			map.put("songpic", songpic);
			map.put("title", title);
			map.put("singer", singer);
			map.put("lyricurl", lyricurl);
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		
		return map;
	}

	private static String xiamidecode(String location) throws UnsupportedEncodingException {
		int _local10;
        int _local2 = Integer.parseInt(location.substring(0, 1));
        String _local3 = location.substring(1, location.length());
        double _local4 = Math.floor(_local3.length() / _local2);
        int _local5 = _local3.length() % _local2;
        String[] _local6 = new String[_local2];
        int _local7 = 0;
        while (_local7 < _local5) {
            if (_local6[_local7] == null) {
                _local6[_local7] = "";
            }
            _local6[_local7] = _local3.substring((((int) _local4 + 1) * _local7),
                    (((int) _local4 + 1) * _local7) + ((int) _local4 + 1));
            _local7++;
        }
        _local7 = _local5;
        while (_local7 < _local2) {
            _local6[_local7] = _local3
                    .substring((((int) _local4 * (_local7 - _local5)) + (((int) _local4 + 1) * _local5)),
                            (((int) _local4 * (_local7 - _local5)) + (((int) _local4 + 1) * _local5))+(int) _local4);
            _local7++;
        }
        String _local8 = "";
        _local7 = 0;
        while (_local7 < ((String) _local6[0]).length()) {
            _local10 = 0;
            while (_local10 < _local6.length) {
                if (_local7 >= _local6[_local10].length()) {
                    break;
                }
                _local8 = (_local8 + _local6[_local10].charAt(_local7));
                _local10++;
            }
            _local7++;
        }
        _local8 = URLDecoder.decode(_local8, "utf8");
        String _local9 = "";
        _local7 = 0;
        while (_local7 < _local8.length()) {
            if (_local8.charAt(_local7) == '^'){
                _local9 = (_local9 + "0");
            } else {
                _local9 = (_local9 + _local8.charAt(_local7));
            };
            _local7++;
        }
        _local9 = _local9.replace("+", " ");
        return _local9;
	}
	
}
```
