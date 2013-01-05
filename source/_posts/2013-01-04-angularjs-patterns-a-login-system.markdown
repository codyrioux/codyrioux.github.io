---
layout: post
title: "AngularJS Patterns: A Login System"
date: 2013-01-04 19:12
comments: true
categories: [AngularJS, JavaScript]
published: false
---
In this post we'll discuss a full fledged authentication system for AngularJS providing a persistent session and allowing the application to display relevant information about the current user.

<!-- more -->

#Motivation
Well it turns out that most of what we know about creating login systems on web pages does not apply to the single page application (SPA) architecture. I am developing an SPA using AngularJS and a handful of other technologies, and it of course required a login system with all the usual bells and whistles: login, logout, registration, persistent sessions, etc... How one might do this using Angular isn't immediately obvious, especially not to a newcomer. I was sadly disappointed after googling I found a very small amount of information on this subject.

One very useful resource that I did find was [Authentication in AngularJS (or similar) based application by Witold Szczerba](http://www.espeo.pl/2012/02/26/authentication-in-angularjs-application) which not only provided a base for this post but introduced me to http interceptors in angular, a very useful feature. However this didn't cover all of the applications needs. This post outlines the full system that I use to manage login and logout functionality in my application.

#Login System

``` javascript services.js
angular.module('MyApp.Services').factory('User', function ($injector, authService) {
    var $http = $http || $injector.get('$http');
    var _this = this;
    this.authenticated = false;
    this.user = null;

    // Check for an active session on the server
    $http.get('/api/user').success(function (data) { 
      if (data) {
      _this.user = data; 
      _this.authenticated = true;
      }
      });

    var isAuthenticated = function () { return _this.authenticated; };
    var isAdministrator = function () { return _this.user && _this.user.isAdministrator; };
    var getUser = function () { return _this.user; };
    var login = function (email, password, callback) {
    $http.post('/api/login', {
username: email,
password: password
}).success(function (data) {
  if (data) {
  _this.user = data;
  _this.authenticated = true;
  }
  return callback(data);
  });
};
var logout = function (callback) {
  if (_this.authenticated) {
    return $http.get('/api/logout', {}, function (data) {
        if (data) {
        this.authenticated = false;
        }
        return callback(data);
        }, 'json');
  } else {
    return callback(false);
  }
};
var register = function (email, password, callback) {
  $http.post('/api/register', {email: email, password: password}).success(function (data) {
      login(email, password, callback);
      });
};

return {
isAuthenticated: isAuthenticated,
                   isAdministrator: isAdministrator,
                   getUser: getUser,
                   login: login,
                   logout: logout,
                   register: register
};
});

```

#Services
More testing to do here...


