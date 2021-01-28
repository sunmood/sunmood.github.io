---
layout: post
title: Model、ModelAndView、ModelMap三者区别
categories:
  - Spring
description: Model、ModelAndView、ModelMap三者区别
keywords: 'SpringMVC'
---

# Model、ModelAndView、ModelMap三者区别

## Model

```
package org.springframework.ui;

import java.util.Collection;
import java.util.Map;

public interface Model {
    Model addAttribute(String var1, Object var2);

    Model addAttribute(Object var1);

    Model addAllAttributes(Collection<?> var1);

    Model addAllAttributes(Map<String, ?> var1);

    Model mergeAttributes(Map<String, ?> var1);

    boolean containsAttribute(String var1);

    Map<String, Object> asMap();
}
```

## ModelAndView

```
package org.springframework.web.servlet;

import java.util.Map;
import org.springframework.ui.ModelMap;
import org.springframework.util.CollectionUtils;

public class ModelAndView {
    private Object view;
    private ModelMap model;
    private boolean cleared = false;

    public ModelAndView() {
    }

    public ModelAndView(String viewName) {
        this.view = viewName;
    }

    public ModelAndView(View view) {
        this.view = view;
    }

    public ModelAndView(String viewName, Map<String, ?> model) {
        this.view = viewName;
        if(model != null) {
            this.getModelMap().addAllAttributes(model);
        }

    }

    public ModelAndView(View view, Map<String, ?> model) {
        this.view = view;
        if(model != null) {
            this.getModelMap().addAllAttributes(model);
        }

    }

    public ModelAndView(String viewName, String modelName, Object modelObject) {
        this.view = viewName;
        this.addObject(modelName, modelObject);
    }

    public ModelAndView(View view, String modelName, Object modelObject) {
        this.view = view;
        this.addObject(modelName, modelObject);
    }

    public void setViewName(String viewName) {
        this.view = viewName;
    }

    public String getViewName() {
        return this.view instanceof String?(String)this.view:null;
    }

    public void setView(View view) {
        this.view = view;
    }

    public View getView() {
        return this.view instanceof View?(View)this.view:null;
    }

    public boolean hasView() {
        return this.view != null;
    }

    public boolean isReference() {
        return this.view instanceof String;
    }

    protected Map<String, Object> getModelInternal() {
        return this.model;
    }

    public ModelMap getModelMap() {
        if(this.model == null) {
            this.model = new ModelMap();
        }

        return this.model;
    }

    public Map<String, Object> getModel() {
        return this.getModelMap();
    }

    public ModelAndView addObject(String attributeName, Object attributeValue) {
        this.getModelMap().addAttribute(attributeName, attributeValue);
        return this;
    }

    public ModelAndView addObject(Object attributeValue) {
        this.getModelMap().addAttribute(attributeValue);
        return this;
    }

    public ModelAndView addAllObjects(Map<String, ?> modelMap) {
        this.getModelMap().addAllAttributes(modelMap);
        return this;
    }

    public void clear() {
        this.view = null;
        this.model = null;
        this.cleared = true;
    }

    public boolean isEmpty() {
        return this.view == null && CollectionUtils.isEmpty(this.model);
    }

    public boolean wasCleared() {
        return this.cleared && this.isEmpty();
    }

    public String toString() {
        StringBuilder sb = new StringBuilder("ModelAndView: ");
        if(this.isReference()) {
            sb.append("reference to view with name '").append(this.view).append("'");
        } else {
            sb.append("materialized View is [").append(this.view).append(']');
        }

        sb.append("; model is ").append(this.model);
        return sb.toString();
    }
}
```

## ModelMap

```
package org.springframework.ui;

import java.util.Collection;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Map.Entry;
import org.springframework.core.Conventions;
import org.springframework.util.Assert;

public class ModelMap extends LinkedHashMap<String, Object> {
    public ModelMap() {
    }

    public ModelMap(String attributeName, Object attributeValue) {
        this.addAttribute(attributeName, attributeValue);
    }

    public ModelMap(Object attributeValue) {
        this.addAttribute(attributeValue);
    }

    public ModelMap addAttribute(String attributeName, Object attributeValue) {
        Assert.notNull(attributeName, "Model attribute name must not be null");
        this.put(attributeName, attributeValue);
        return this;
    }

    public ModelMap addAttribute(Object attributeValue) {
        Assert.notNull(attributeValue, "Model object must not be null");
        return attributeValue instanceof Collection && ((Collection)attributeValue).isEmpty()?this:this.addAttribute(Conventions.getVariableName(attributeValue), attributeValue);
    }

    public ModelMap addAllAttributes(Collection<?> attributeValues) {
        if(attributeValues != null) {
            Iterator var2 = attributeValues.iterator();

            while(var2.hasNext()) {
                Object attributeValue = var2.next();
                this.addAttribute(attributeValue);
            }
        }

        return this;
    }

    public ModelMap addAllAttributes(Map<String, ?> attributes) {
        if(attributes != null) {
            this.putAll(attributes);
        }

        return this;
    }

    public ModelMap mergeAttributes(Map<String, ?> attributes) {
        if(attributes != null) {
            Iterator var2 = attributes.entrySet().iterator();

            while(var2.hasNext()) {
                Entry<String, ?> entry = (Entry)var2.next();
                String key = (String)entry.getKey();
                if(!this.containsKey(key)) {
                    this.put(key, entry.getValue());
                }
            }
        }

        return this;
    }

    public boolean containsAttribute(String attributeName) {
        return this.containsKey(attributeName);
    }
}
```