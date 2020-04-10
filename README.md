# MovieSearch
네이버api를이용하여 영화정보검색을 해보았습니다.

# Functions
> 주요기능
> - Restful 이용하여 영화 목록을 json으로 얻어와 파싱하였습니다.
> - client id, client secret 이 존재하기 때문에 DI를 사용하였습니다.
> - 특수문자입력이 확인되면 검색이 불가하도록 하였습니다.

# server
> http://112.148.190.213:8090/lookup/index.jsp
# source

## controller
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

import com.movie.client.service.searchService;
import com.nhncorp.lucy.security.xss.XssFilter;

@RestController
public class searchContoller {
	@Autowired
	private searchService searchService;

	private static final int display = 5;

	@RequestMapping(value="search.do",method= {RequestMethod.GET,RequestMethod.POST})
	public ModelAndView SearchMovies(@RequestParam("v") String value, @RequestParam("page") int page) throws Exception {
		ModelAndView mnv = new ModelAndView();

		XssFilter filter = XssFilter.getInstance("lucy-xss-superset.xml");
		if(filter.doFilter(value).equals(value)) {
			mnv.setViewName("resultView.jsp");
			mnv.addObject("list",searchService.getDatas(value,page,display));
		}else {
			mnv.setViewName("redirect:checkText.jsp");
		}
		return mnv;
	}
}
```

## service
```

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.util.HashMap;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Service;

import com.movie.client.dao.SecretClient;

@Service
public class searchService {
	static final int pageloader = 10;
	public HashMap<String, Object> getDatas(String value, int page, int display) {
		HashMap<String, Object> map = new HashMap<String, Object>();
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/Secret/OpenAPI.xml");
		SecretClient  sc = (SecretClient)ctx.getBean("secret");

		int start = (int)((page-1) * display) + 1;
		try {
			String text = URLEncoder.encode(value,"utf-8");
			String apiURL = "https://openapi.naver.com/v1/search/movie.json?query=" + text + "&display=" + display+"&start="+start;
			URL url = new URL(apiURL);
			HttpURLConnection con = (HttpURLConnection) url.openConnection();
            con.setRequestMethod("GET");
            con.setRequestProperty("X-Naver-Client-Id", sc.getKey());
            con.setRequestProperty("X-Naver-Client-Secret", sc.getSecret());
            int responseCode = con.getResponseCode();
            BufferedReader br;
            if(responseCode == 200) {
            	br = new BufferedReader(new InputStreamReader(con.getInputStream()));
            }else {
            	br = new BufferedReader(new InputStreamReader(con.getErrorStream()));
            }
            StringBuilder sb = new StringBuilder();
            String read;
            while((read = br.readLine()) != null) {
            	sb.append(read + "\n");
            }
            br.close();
            con.disconnect();
            String data = sb.toString();
            JSONParser p = new JSONParser();
            JSONObject Jobj = (JSONObject)p.parse(data);
            JSONArray Jarray = (JSONArray) Jobj.get("items");


            long total = (long)Jobj.get("total");
            int MaxPage = (int) (total / display);
            if(MaxPage % display > 0 ) {
            	MaxPage++;
            }
            int lastsection = ((int)((MaxPage - 1) / pageloader));
            int nowsection = ((int)((page - 1) / pageloader));

            map.put("nowsection",nowsection);
            map.put("lastsection", lastsection);
            map.put("on", page);
            map.put("maxPage", MaxPage);
            map.put("title",value);
            map.put("total",Jobj.get("total"));
            map.put("list",Jarray);
            System.out.println(data);
		}catch(Exception e) {
			e.printStackTrace();
		}
		return map;
	}

}
```

# 결과이미지
## 메인
![main](image/main.PNG)
## 결과
![result](image/result.PNG)

