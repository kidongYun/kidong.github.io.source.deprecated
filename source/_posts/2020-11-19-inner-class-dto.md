---
layout: post
title:  "Inner Class Style for DTO"
date:   2020-11-16 11:38:54 +0900
categories: java
---

```java

    @RequestMapping(value = "/penalty", method = { RequestMethod.GET, RequestMethod.POST }, produces = {"application/json;charset=UTF-8"})
    public ResponseEntity<Response<PenaltyRes>> penalty(PenaltyReq req) {
        try {
            return new ResponseEntity<>(new Response.Builder<PenaltyRes>()
                    .errorCode(HttpStatus.OK.value())
                    .errorDesc(HttpStatus.OK.getReasonPhrase())
                    .data(mypageCancelService.penalty(req)).build(), HttpStatus.OK);

        } catch (Exception e) {
            e.printStackTrace();
            return new ResponseEntity<>(new Response.Builder<PenaltyRes>()
                    .errorCode(HttpStatus.INTERNAL_SERVER_ERROR.value())
                    .errorDesc(e.getMessage())
                    .build(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

```