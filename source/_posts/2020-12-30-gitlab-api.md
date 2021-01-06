---
layout: post
title:  "Gitlab Api 사용하기"
date:   2020-12-30 20:00:00 +0900
categories: java
---

```java
package com.air.interpark.controller;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import lombok.extern.slf4j.Slf4j;
import net.rcarz.jiraclient.*;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

import java.io.IOException;
import java.net.InetAddress;
import java.net.URI;
import java.net.URISyntaxException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;

@Slf4j
@Controller
public class AutoDeployController {

    @RequestMapping(value = "/deploy", method = {RequestMethod.POST, RequestMethod.GET}, produces = {"text/html;charset=UTF-8", "application/xml;charset=UTF-8", "application/json;charset=UTF-8"})
    public ResponseEntity<?> deploy(@RequestParam String pmsNo) throws URISyntaxException, IOException, JiraException {

        URI mergeRequestUri = new URIBuilder(new URI("http://gitlab.interpark.com/api/v4/projects/1090/merge_requests"))
                .addParameter("search", "ITSERVICE-39158").addParameter("state", "opened").build();
        List<MergeRequest> mergeRequests = new ObjectMapper().readValue(EntityUtils.toString(call(mergeRequestUri)), new TypeReference<List<MergeRequest>>() {});

        URI versionUri = new URIBuilder(new URI("http://gitlab.interpark.com/api/v4/projects/1090/merge_requests/" + mergeRequests.get(0).getIid() + "/versions")).build();
        List<Version> versions = new ObjectMapper().readValue(EntityUtils.toString(call(versionUri)), new TypeReference<List<Version>>() {});

        URI versionDetaionUri = new URIBuilder(new URI("http://gitlab.interpark.com/api/v4/projects/1090/merge_requests/" + mergeRequests.get(0).getIid() + "/versions/" + versions.get(0).getId())).build();
        VersionDetail versionDetail = new ObjectMapper().readValue(EntityUtils.toString(call(versionDetaionUri)), VersionDetail.class);

        List<Diff> diffs = versionDetail.getDiffs();

        /* JIRA 계정 로그인 및 연동 */
        BasicCredentials creds = new BasicCredentials("[ID]", "[PASSWORD]");
        JiraClient jira = new JiraClient("https://pms.interpark.com/", creds);

        /* JIRA ISSUE 생성 */
        Issue issue = jira.createIssue("DEP", "수시반영 형상관리")
                .field("customfield_10544", "정규반영")
                .field(Field.SUMMARY, "summary")
                .field(Field.DESCRIPTION, "description")
                .execute();

        return ResponseEntity.status(HttpStatus.OK).body(HttpStatus.OK.getReasonPhrase());
    }

    private HttpEntity call(URI uri) throws IOException {
        HttpGet httpGet = new HttpGet(uri);
        httpGet.addHeader("PRIVATE-TOKEN", "[PRIVATE TOKEN]");
        HttpResponse response = HttpClientBuilder.create().build().execute(httpGet);
        return response.getEntity();
    }

    @Getter
    @Setter
    @ToString
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class MergeRequest {
        @JsonProperty(value = "iid")
        private String iid;
    }

    @Getter
    @Setter
    @ToString
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class Version {
        @JsonProperty(value = "id")
        private String id;
    }

    @Getter
    @Setter
    @ToString
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class VersionDetail {
        @JsonProperty(value = "diffs")
        private List<Diff> diffs;
    }

    @Getter
    @Setter
    @ToString
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class Diff {
        @JsonProperty(value = "new_path")
        private String newPath;
    }
}

```