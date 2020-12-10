---
layout: post
title:  "HttpClient"
date:   2020-12-04 11:38:54 +0900
categories: spring
---

```java

package com.interpark.tour.air.api.product.service.domestic;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.interpark.tour.air.api.product.common.Utility;
import com.interpark.tour.air.api.product.model.domestic.dto.AirlineControlTimer;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.cookie.BasicClientCookie;
import org.apache.http.util.EntityUtils;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.net.URI;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Slf4j
@Service(value = "AirlineControlTimerService")
public class AirlineControlTimerService {

    /** DOM_AIRLINE_CONTROL_HISTROY 테이블에 스케줄, 마이페이지 ON/OFF 내역 등록 */
    public AirlineControlTimer.Res registerHistory(AirlineControlTimer.Req req) throws Exception {
        URI uri = new URIBuilder(new URI("http://batchadmin.interparktour.com/admin/airControlTimerAdd.do"))
                .addParameter("searchAirline", "AL")
                .addParameter("airline", req.getAirline())
                .addParameter("startDateTime", req.getStartDateTime().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm")))
                .addParameter("endDateTime", req.getEndDateTime().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm")))
                .addParameter("inputDate", LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm")))
                .addParameter("inputId","SYSTEM")
                .addParameter("scheduleOff", req.getScheduleOff())
                .addParameter("mypageOff", req.getMypageOff())
                .build();

        AirlineControlTimer.Res res = call(uri);

        int HISTORY_ADD_SUCCESS = 101;
        if(HISTORY_ADD_SUCCESS != res.getErrorCode()) {
            throw new Exception("히스토리 테이블에 새 내역을 추가하는 중 오류가 발생했습니다");
        }

        return res;
    }

    /** 배치 스케줄러에 스케줄, 마이페이지 ON/OFF 등록 */
    public AirlineControlTimer.Res registerBatch(AirlineControlTimer.Req req) throws Exception {
        URI uri = new URIBuilder(new URI("http://batchadmin.interparktour.com/admin/airControlTimerUpdateBatch.do"))
                .addParameter("no", null)
                .addParameter("airline", req.getAirline())
                .addParameter("startDateTime", req.getStartDateTime().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm")))
                .addParameter("endDateTime", req.getEndDateTime().format(DateTimeFormatter.ofPattern("yyyyMMddHHmm")))
                .addParameter("scheduleOff", req.getScheduleOff())
                .addParameter("mypageOff", req.getMypageOff())
                .addParameter("type", "add")
                .build();

        AirlineControlTimer.Res res = call(uri);

        int BATCH_ADD_SUCCESS = 104;
        if(BATCH_ADD_SUCCESS != res.getErrorCode()) {
            throw new Exception("배치 테이블에 새 내역을 추가하는 중 오류가 발생했습니다");
        }

        return res;
    }

    public AirlineControlTimer.Res register(AirlineControlTimer.Req req) throws Exception {
        registerHistory(req);
        return registerBatch(req);
    }

    private AirlineControlTimer.Res call(URI uri) throws IOException {
        HttpClient httpClient = HttpClientBuilder.create().setDefaultCookieStore(cookieConfig("N19147")).build();

        HttpGet httpGet = new HttpGet(uri);
        HttpResponse response = httpClient.execute(httpGet);
        HttpEntity entity = response.getEntity();

        return new ObjectMapper().readValue(EntityUtils.toString(entity), AirlineControlTimer.Res.class);
    }

    private BasicCookieStore cookieConfig(String adminId) {
        BasicClientCookie adminIdCookie = new BasicClientCookie("ADMIN_ID", adminId);
        adminIdCookie.setDomain("batchadmin.interparktour.com");
        adminIdCookie.setPath("/");

        BasicClientCookie awpKeyCookie = new BasicClientCookie("AWP_KEY", Utility.getEncSHA256("interpark_" + adminId + "_airadmin12#$"));
        awpKeyCookie.setDomain("batchadmin.interparktour.com");
        awpKeyCookie.setPath("/");

        BasicCookieStore cookieStore = new BasicCookieStore();
        cookieStore.addCookie(adminIdCookie);
        cookieStore.addCookie(awpKeyCookie);

        return cookieStore;
    }
}


```