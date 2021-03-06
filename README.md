# Code-Sample-Guide


## Express examples
[WillCallTickets - Server.js](https://github.com/WillCallTickets/willcall/blob/master/server.js)  
[Example routing file](https://github.com/WillCallTickets/WillCall/blob/master/routes/stripe.js)


## Promise examples
[updating an invoice in the order flow](https://github.com/WillCallTickets/willcall/blob/master/server/controllers/store.js)  
[using new Promise() - line 183](https://github.com/WillCallTickets/WillCall/blob/master/server/controllers/members.js)  
[using angular's $q -  line 61](https://github.com/WillCallTickets/willcall/blob/master/public/js/models/showModel.js)  
[Use of promses.all - line 50](https://github.com/WillCallTickets/WillCall/blob/master/lib/dbops/invoices.js)   
[Another example of promises.all - line 257](https://github.com/WillCallTickets/blazecast/blob/master/controllers/api.js)
```
// line 257

exports.getFedPodcastEpisodes = function(req, res, next){

  var podcast = {};
  var pId = req.params.itunes_podcast_id;

  // eCollection is the returned collection of episode objects

  return audiosearch.get('/shows', {'itunes_id':pId})
  .then(function(data){
    podcast = data;
    var episodePromises = [];
    if(podcast.episode_ids.length > 0){
      podcast.episode_ids.forEach(function(itm){
        episodePromises.push( audiosearch.getEpisode(itm) );
      });
    }
    return Promise.all(episodePromises);
  })
  .then(function(data){
    podcast.eCollection = data;
    res.send(podcast);
  })
  .catch(function(err){
    console.log('ERROR AT API CATCH', err);
    res.send(err);
  });

}
```
## Recent project
[REACT crud with Auth0 and react-router@v4](https://github.com/WillCallTickets/react-auth0-routerv4)  
Currently a work-in-progress
Built using code and concepts from [Rem Zolotykh](https://remzolotykh.net/crud-with-redux-01-application-and-redux-setup/)  
Integration of Auth0 for social logins and user mgmt handling [Auth0](https://auth0.com/)   
&nbsp;&nbsp; - refactored the Auth0 implementation to work with react-router@v4
  

## Galvanize projects
[Q2 group project - Cloak And Tagger](https://github.com/WillCallTickets/cloakandtagger)  
Team project highlighting a git centralized workflow  
Utilized Google Maps, Microsoft Project Oxford, Cloudinary and oAuth Apis  
  
[Q3 group project - Blazecast](https://github.com/WillCallTickets/blazecast)  
Built using Angularjs, social logins, JWTs, Audiosear.ch and iTunes search Apis  
Team project emphasizing communication and git feature-branch workflow  
  
[Q4 solo "Capstone" project - WillCallTicketing](https://github.com/WillCallTickets/WillCall)  
[WillCallTicketing - Live Link](https://willcalltickets.herokuapp.com/)  
Written with the NEAP stack – Node.js, Express, Angular and PostgreSql  
Employed Stripe Connect Api to onboard members and reduce vetting concerns  
Used Stripe Payments Api to instill customer confidence and eliminate PCI/DSS concerns § Business model underscores all-in-pricing, simplicity of member sign up and security  
  

## RE: "this"
Historically, my methodology for handling "this" was to be explicit when referencing, defining it as "self" at the head of a function. My mantra here is "When there is a possibiliity of ambiguity - be specific".  
["this" reference example](https://github.com/WillCallTickets/Fox_2014/blob/master/Z2Web/assets/javascripts/z2ModalService.js)  
```
...
    $.fn.z2Modal = function (_fnHandlerPath, _fnSuccess, _inputs, _fnParamValidate) {  
  
        //ensure we are just dealing with a single element
        var self = $(this).get(0);
        if (self != undefined) {
            var selfId = self.id;
            var sender = $('#' + selfId + ' .wct-modal-action');
            var errorDisplayElement = $('#' + selfId + ' .wctmodal-error');
            var successDisplayElement = $('#' + selfId + ' .wctmodal-success');
...
```
##### More recently, I had an issue with "this", realized the mistake quickly and fixed it using the same pattern
```
// before 
  login() {
    this.lock.show({});
    
    return {
      hide() {
        this.lock.hide();
      }
    }
  }
  
// after
  login() {
    let self = this;
    self.lock.show({});
    
    return {
      hide() {
        self.lock.hide();
      }
    }
  }
```
#### Other ways to handle *this*
```
.call([thisDeclaration], [explicitly listed arg], [explicitly listed arg], ...)  
.apply([thisDeclaration], [array of args])  
.bind([thisDeclaration])  
someFunction.bind([thisDeclaration])  
```
   
   
In React, when using es6 classes, class methods are not auto-bound by default, *this* is undefined. There are a few methods to fix this issue:  
1) bind in the element render - `<button onClick={this.clickHandler.bind(this)}>`  
2) bind in the constructor - `this.clickHandler = this.clickHandler.bind(this);` * **recommended** *  
3) use an arrow function in the render - `<button onClick={(e) => this.clickHandler(e)}>`  
4) Use React.createClass - binds this to the component automatically  
5) Use arrow function in class property `clickHandler = () => {...}` - this requires some other setup but may become preferred in the future  
  
  
## A javascript "elegant" solution 
handling http/https issues with loading static resources
```
//Server.js

// Proxy Resource - resolves by correcting https and http for resources
app.get('/proxyresource/:resourceurl',
  resourceController.proxyResource);


// resourceController
var request = require('request');
var atob = require('atob');

exports.proxyResource = function(req, res, next) {
  
  // decode
  var decodedString = atob(req.params.resourceurl);
  // send it back as a stream
  // console.log('PXY API atob', decodedString, req.params.resourceurl);
  
  request(decodedString).pipe(res);
};
```
##### use as a filter to wrap the link
```
angular.module('MyApp')
  .filter('removeProtocol', function () {
    return function (input) {
      if (input) {
        return input.replace(/^http:\/\//g, '')
        .replace(/^https:\/\//g, '');
      }
    }
  })
  .filter('proxyResource', function () {
    return function (input) {
      if (input) {
        var encoded = btoa(input);
        return '/proxyresource/' + encoded;
      }
    }
  })
  .filter('trustMe', ['$sce', function($sce){
    return function(input){
      if(input){
        return $sce.trustAsHtml(input);
      }
    }
  }]);
  
  
  //usage
  
  // videogular only uses src and type attribs - normalize episode to source
  this.convertEpisodeToSource = function(episode){
    var source = {};
    source.src = $sce.trustAsResourceUrl(proxyResourceFilter(episode.audio_url));
    source.type = episode.type;
    return source;
  };
  
  ```
  
  

## A c# "elegant" solution

*** the ORM that I used, subsonic, created an active record pattern for interacting with the database ***

[_Collection implementation using generics](https://github.com/WillCallTickets/Fox_2014/blob/master/Utils/_Collection.cs)

* The algorithm could certainly be refactored, but these collections were generally 2 - 20 members. The emphasis was on handling several different tables in a like manner.  
* Solved the problem of having several tables/classes that needed to share the same functionality.  
* Address the issue of managing collections that need to keep track of a user-defined ordering of rows. 
* [Table example utilizing the iDisplayOrder column](https://github.com/WillCallTickets/Fox_SqlProgrammability/blob/master/Tables/dbo.JShowAct.Table.sql)
* I was able to expand on this model and implement different sort orders based on "principal" ownership, allowing for multiple discrete entites to have their own sort-order over a shared table
* This allowed me to decouple the manipulation layer (jquery ajax calls) from the storage layer (sql). 	
* All of the shared functionality could be accessed from a single file for management of several tables  
[PrincipalOrder.ashx](https://github.com/WillCallTickets/Fox_2014/blob/master/WcWeb/Admin/_customControls/PrincipalOrder.ashx)

