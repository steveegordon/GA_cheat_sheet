# FIREBASE
### by Claude and Eunice

----

# Setting up and syncing a firebase array in your controller.

#####In HTML file, include scripts:
 
 	<script src="js/angular.js"></script>
    <script src="https://cdn.firebase.com/js/client/2.2.1/firebase.js"></script>
    <script src="https://cdn.firebase.com/libs/angularfire/1.0.0/angularfire.min.js">
    
    
#####In the main module, (i.e. app.js) add firebase dependency:
 
 	angular.module('app', ['firebase']);
    
    
#####Inject firebase to the controller & sync data into $firebaseArray argument:
 
 	 angular
        .module('app')
        .controller('appController',appController);
        
        appController.$inject = ['$firebaseArray'];
        
#####Pass the firebase array argment to the controller function:
 
 	 function appController($firebaseArray)
 	 
##### In the controller function, create a reference variable that points to the firebase data to access:

	var ref = new Firebase("https://angularfirebaseapp.firebaseio.com/appdata/");