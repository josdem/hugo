+++
categories = ["techtalk", "code"]
date = "2016-09-06T21:06:21-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "grails", "spring security","login flow"]
title = "Spring Security & Login Functionality"

+++

Let's use the [Spring Security Core](https://grails-plugins.github.io/grails-spring-security-core/) to create login functionality Specification.

First we need to add spring security core to our `build.gradle` file

```groovy
compile 'org.grails.plugins:spring-security-core:3.1.1'
```

Then enter to the interactive Grails console, in the project home type: `grails`. Using `s2-quickstart` command you can create a domain class for the Spring Security plugin, in this case we're going to create an User domain class name used to Spring security to authenticate users with an specific Role.

```bash
s2-quickstart com.jos.dem.vetlog User Role
```

With previous action Grails will create the following domain classes under `com.jos.dem.vetlog` package

**User.groovy**

```groovy
package com.jos.dem.vetlog

import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString

@EqualsAndHashCode(includes='username')
@ToString(includes='username', includeNames=true, includePackage=false)
class User implements Serializable {

  private static final long serialVersionUID = 1

  transient springSecurityService

  String username
  String password
  boolean enabled = true
  boolean accountExpired
  boolean accountLocked
  boolean passwordExpired

  Set<Role> getAuthorities() {
    UserRole.findAllByUser(this)*.role
  }

  def beforeInsert() {
    encodePassword()
  }

  def beforeUpdate() {
    if (isDirty('password')) {
      encodePassword()
    }
  }

  protected void encodePassword() {
    password = springSecurityService?.passwordEncoder ? springSecurityService.encodePassword(password) : password
  }

  static transients = ['springSecurityService']

  static constraints = {
    password blank: false, password: true
    username blank: false, unique: true
  }

  static mapping = {
    password column: '`password`'
  }
}
```

**Role.groovy**

```groovy
package com.jos.dem.vetlog

import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString

@EqualsAndHashCode(includes='authority')
@ToString(includes='authority', includeNames=true, includePackage=false)
class Role implements Serializable {

  private static final long serialVersionUID = 1

  String authority

  static constraints = {
    authority blank: false, unique: true
  }

  static mapping = {
    cache true
  }
}
```

**UserRole.groovy**

```groovy
package com.jos.dem.vetlog

import grails.gorm.DetachedCriteria
import groovy.transform.ToString

import org.apache.commons.lang.builder.HashCodeBuilder

@ToString(cache=true, includeNames=true, includePackage=false)
class UserRole implements Serializable {

  private static final long serialVersionUID = 1

  User user
  Role role

  @Override
  boolean equals(other) {
    if (other instanceof UserRole) {
      other.userId == user?.id && other.roleId == role?.id
    }
  }

  @Override
  int hashCode() {
    def builder = new HashCodeBuilder()
    if (user) builder.append(user.id)
    if (role) builder.append(role.id)
    builder.toHashCode()
  }

  static UserRole get(long userId, long roleId) {
    criteriaFor(userId, roleId).get()
  }

  static boolean exists(long userId, long roleId) {
    criteriaFor(userId, roleId).count()
  }

  private static DetachedCriteria criteriaFor(long userId, long roleId) {
    UserRole.where {
      user == User.load(userId) &&
      role == Role.load(roleId)
    }
  }

  static UserRole create(User user, Role role) {
    def instance = new UserRole(user: user, role: role)
    instance.save()
    instance
  }

  static boolean remove(User u, Role r) {
    if (u != null && r != null) {
      UserRole.where { user == u && role == r }.deleteAll()
    }
  }

  static int removeAll(User u) {
    u == null ? 0 : UserRole.where { user == u }.deleteAll()
  }

  static int removeAll(Role r) {
    r == null ? 0 : UserRole.where { role == r }.deleteAll()
  }

  static constraints = {
    role validator: { Role r, UserRole ur ->
      if (ur.user?.id) {
        UserRole.withNewSession {
          if (UserRole.exists(ur.user.id, r.id)) {
            return ['userRole.exists']
          }
        }
      }
    }
  }

  static mapping = {
    id composite: ['user', 'role']
    version false
  }
}
```

Next let's create a `Profile` entity to the User domain class, it is a good practice you can gather user information in a separate class from User such as email, name, lastname, etc.

```groovy
package com.jos.dem.vetlog

class Profile {
  String name
  String lastName
  String email

  static belongsTo = [User]

  static constraints = {
    name blank:false,size:1..100
    lastName blank:false,size:1..100
    email blank:false,email:true,unique:true,size:6..200
  }
}
```

Don't forget to add Profile reference to the User domain.

```groovy
package com.jos.dem.vetlog

import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString

@EqualsAndHashCode(includes='username')
@ToString(includes='username', includeNames=true, includePackage=false)
class User implements Serializable {

  private static final long serialVersionUID = 1

  transient springSecurityService

  String username
  String password
  boolean enabled = true
  boolean accountExpired
  boolean accountLocked
  boolean passwordExpired

  Profile profile

  Set<Role> getAuthorities() {
    UserRole.findAllByUser(this)*.role
  }

  def beforeInsert() {
    encodePassword()
  }

  def beforeUpdate() {
    if (isDirty('password')) {
      encodePassword()
    }
  }

  protected void encodePassword() {
    password = springSecurityService?.passwordEncoder ? springSecurityService.encodePassword(password) : password
  }

  static transients = ['springSecurityService']

  static constraints = {
    password blank: false, password: true
    username blank: false, unique: true
  }

  static mapping = {
    password column: '`password`'
  }
}
```

Now, use `${PROJECT_HOME}/grails-app/init/Bootstrap.groovy` to create a default User, Role and UserRole

```groovy
import grails.util.Environment

import com.jos.dem.vetlog.User
import com.jos.dem.vetlog.Profile
import com.jos.dem.vetlog.Role
import com.jos.dem.vetlog.UserRole

class BootStrap {

  def init = { servletContext ->
    createUserWithRole('josdem', '12345678', 'joseluis.delacruz@gmail.com', 'ROLE_USER')
  }

  def createUserWithRole(String username, String password, String email, String authority) {
    if(Environment.current != Environment.PRODUCTION){
      def role = new Role(authority:authority).save()
      def user = new User(username:username,
        password:password,
        enabled:true,
        profile:new Profile(name:username, lastName:'lastName', email:email)).save()
      new UserRole(user:user, role:role).save()
    }
  }

  def destroy = {
  }

}
```

Then let's create a custom login page at `${PROJECT_HOME}/grails-app/views/login/auth.gsp`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="layout" content="home">
    <asset:stylesheet src="third-party/vetlog-theme/css/style.css" />
    <asset:stylesheet src="third-party/vetlog-theme/css/plugins.css" />
    <asset:stylesheet src="third-party/vetlog-theme/css/demo.css" />
  </head>
  <body class="login">

    <div class="container">
      <div class="row">
        <div class="col-md-4 col-md-offset-4">
          <div class="login-banner text-center">
            <h1>
              <asset:image src="third-party/vetlog-theme/img/flex-admin-logo.png" />
            </h1>
          </div>
          <div class="portlet portlet-green">
            <div class="portlet-heading login-heading">
              <div class="portlet-title">
                <h4>${message(code:'login.welcome')}</h4>
              </div>
              <div class="clearfix"></div>
            </div>
            <div class="portlet-body">
              <g:if test="${flash.message || session.message}">
              <p class="text-info">${flash.message ?: session.message}</p>
              </g:if>
              <form action='${postUrl}' method='POST' id='loginForm' class='cssform' autocomplete='off'>
                <fieldset>
                  <label for="username">${message(code:'login.username')}</label>
                  <input type="text" name='username' class="form-control" placeholder="Nombre de usuario" id='username' value="${flash.username ?: session.username}" >
                  <label for="password">${message(code:'login.password')}</label>
                  <input type="password" name='password' class="form-control" placeholder="Contraseña" id='password' >
                  <br/>
                  <button id="btn-success" type="submit" class="btn btn-lg btn-primary btn-block">${message(code:'login.action')}</button>
                  <hr>
                  <g:link controller="user" action="create" class="btn btn-block btn-default">${message(code:'login.register')}</g:link>
                </fieldset>
                <br>
                <p class="small">
                <g:link controller="recovery" action="forgotPassword">${message(code:'login.forgot')}</g:link>
                </p>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </body>
</html>
```

The Spring Security plugin maintains its configuration in `${PROJECT_HOME}/grails-app/conf/application.groovy`

```groovy
// Added by the Spring Security Core plugin:
grails.plugin.springsecurity.userLookup.userDomainClassName = 'com.jos.dem.vetlog.User'
grails.plugin.springsecurity.userLookup.authorityJoinClassName = 'com.jos.dem.vetlog.UserRole'
grails.plugin.springsecurity.authority.className = 'com.jos.dem.vetlog.Role'
grails.plugin.springsecurity.successHandler.defaultTargetUrl = '/dashboard'
grails.plugin.springsecurity.controllerAnnotations.staticRules = [
  [pattern: '/',               access: ['permitAll']],
  [pattern: '/error',          access: ['permitAll']],
  [pattern: '/index',          access: ['permitAll']],
  [pattern: '/home/index',     access: ['permitAll']],
  [pattern: '/index.gsp',      access: ['permitAll']],
  [pattern: '/shutdown',       access: ['permitAll']],
  [pattern: '/assets/**',      access: ['permitAll']],
  [pattern: '/login/**',       access: ['permitAll']],
  [pattern: '/**/js/**',       access: ['permitAll']],
  [pattern: '/**/css/**',      access: ['permitAll']],
  [pattern: '/**/images/**',   access: ['permitAll']],
  [pattern: '/**/favicon.ico', access: ['permitAll']],
  [pattern: '/dashboard/**',   access: ['IS_AUTHENTICATED_FULLY']],
  [pattern: '/logoff/**',      access: ['IS_AUTHENTICATED_FULLY']]
]

grails.plugin.springsecurity.filterChain.chainMap = [
  [pattern: '/assets/**',      filters: 'none'],
  [pattern: '/**/js/**',       filters: 'none'],
  [pattern: '/**/css/**',      filters: 'none'],
  [pattern: '/**/images/**',   filters: 'none'],
  [pattern: '/**/favicon.ico', filters: 'none'],
  [pattern: '/**',             filters: 'JOINED_FILTERS']
]
```

As you can see we have a default page the user is redirected when he/she authenticates, in another hand we establish permitAll or is authenticated fully to specific URLs.

Finally create an DashboardController and dashboard index to redirect to the user authenticated

```groovy
package com.jos.dem.vetlog

class DashboardController {
  def index() {}
}
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <h1>hello user!</h1>
  </body>
</html>
```

To run the project in ${PROJECT_HOME} type:

```bash
grails
```

Go to `http://localhost:8080`

To download the project:

```bash
git clone https://github.com/josdem/vetlog.git
git fetch
git checkout feature/8
```

[Return to the main article](/techtalk/grails)
