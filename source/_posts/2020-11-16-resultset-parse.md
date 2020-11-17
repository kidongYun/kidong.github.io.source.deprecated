---
layout: post
title:  "Java DBResultSet Parsing"
date:   2020-11-16 11:38:54 +0900
categories: java
---


```java

import com.fasterxml.jackson.databind.ObjectMapper;
import java.sql.SQLException;
import java.util.*;

public class ResultSetUtil {

    /** DBResultSet -> List<Map<String, String>> */
    public static List<Map<String, String>> toMap(DBResultSet rset) throws SQLException {
        List<Map<String, String>> rows = new ArrayList<>();
        rset.initRow();

        while (rset.next()) {
            Map<String, String> column = new HashMap<>();
            for(int i$ = 1; i$ <= rset.columnCount; i$++) {
                column.put(rset.getColumnName(i$), rset.getString(i$));
            }
            rows.add(column);
        }

        return rows;
    }

    /** List<Map<String, String>> -> List<T> */
    public static <T> List<T> toDomain(List<Map<String, String>> maps, Class<T> clazz) {
        List<T> objects = new ArrayList<>();
        for(Map<String, String> map : maps) {
            objects.add(new ObjectMapper().convertValue(map, clazz));
        }

        return objects;
    }

    /** DBResultSet -> List<T> */
    public static <T> List<T> toDomain(DBResultSet rset, Class<T> clazz) throws SQLException {
        return toDomain(toMap(rset), clazz);
    }

    /** List<Map<String, String>> -> DBResultSet */
    public static DBResultSet toResultSet(List<Map<String, String>> rows) throws SQLException {
        Set<String> columnNames = rows.get(0).keySet();
        EditableResultSet ers = new EditableResultSet(columnNames.toArray(new String[0]));

        for(Map<String, String> row : rows) {
            ers.absolute(ers.addRow());
            for(String columnName : columnNames) {
                ers.setValue(columnName, row.get(columnName));
            }
        }

        return ers;
    }

    /** List<T> -> DBResultSet */
    public static <T> DBResultSet parseResultSet(List<T> rows) throws SQLException {
        return toResultSet(toMap(rows));
    }

    /** List<T> -> List<Map<String, String>> */
    @SuppressWarnings("unchecked")
    public static <T> List<Map<String, String>> toMap(List<T> rows) throws SQLException {
        List<Map<String, String>> maps = new ArrayList<>();

        for(T row : rows) {
            maps.add(new ObjectMapper().convertValue(row, Map.class));
        }

        return maps;
    }
}

```