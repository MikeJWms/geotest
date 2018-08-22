<template>
  <div class="hello">
    <button v-on:click="loadLocations">+1000 documents to collection</button>
    <button v-on:click="findPoints">Find Points</button>
    <p>{{ this.clientPosition }}</p>

    <div v-if="true">
      <ul v-for="location in locations" :key=location.g>
        <li class="location">
          <p>{{ location.d }}</p>
          <p>{{ location.g }}</p>
          <p>Distance: {{findDistance(location.l._lat, location.l._long)}}</p>

        </li>
      </ul>
    </div>

  </div>
</template>

<script>
import firebase from 'firebase'
import 'firebase/firestore' //if use firestore
import { GeoFirestore } from 'geofirestore'
import axios from 'axios'

if (!firebase.apps.length) {
  firebase.initializeApp({
    apiKey: "XXXX",
    authDomain: "XXXX",
    databaseURL: "XXXX",
    projectId: "XXXX",
    storageBucket: "XXXX",
    messagingSenderId: "XXXX"
  })
}

firebase.firestore().settings({ timestampsInSnapshots: true })
const db = firebase.firestore()

// Create a Firebase reference where GeoFirestore will store its information
const collectionRef = firebase.firestore().collection('geo_test');

// Create a GeoFirestore index
const geoFirestore = new GeoFirestore(collectionRef);
export default {
  data() {
    return {
      clientPosition:{
        lat: null,
        lng: null
      },
      locations: []
    }
  },
  name: 'HelloWorld',
  props: {
    msg: String
  },
  beforeMount: function() {
    if(navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          console.log(`here is the position lat: ${position.coords.latitude}`)
          console.log(`here is the position lng: ${position.coords.longitude}`)
          this.clientPosition.lat = position.coords.latitude
          this.clientPosition.lng = position.coords.longitude
        })
      } else {
        alert('Sorry, this browser does not support location, or you have blocked location from Scout Military.')
      }

    fetch('http://ip-api.com/json')
    .then((response) => {
      return response.json();
    })
    .then((myJson) => {
      this.clientPosition.lat = myJson.lat
      this.clientPosition.lng = myJson.lon
    });

  },
  methods: {
    findDistance (lat,lng) {
      const _location1 = new firebase.firestore.GeoPoint(this.clientPosition.lat, this.clientPosition.lng)
      const _location2 = new firebase.firestore.GeoPoint(lat, lng)
      return GeoFirestore.distance(_location1, _location2)
    },
    setLocation () {
      
    },
    loadLocations (evt) {
      evt.preventDefault()
      for(let i = 0; i < 1000; i ++){
        // place random geopoints around your area within a range of ~500 miles (+-7 degrees)
        let rand_lat = (Math.random() * 14) - 7
        let rand_lng = (Math.random() * 14) - 7
        geoFirestore.add({ 
        coordinates: new firebase.firestore.GeoPoint(this.clientPosition.lat + rand_lat, this.clientPosition.lng + rand_lng)
        }).then((docRef) => {
        console.log(docRef.id); // ID of newly added document
      }, (error) => {
        console.log('Error: ' + error);
      });
      }
    },
    geoTestMethod (evt) { //adds one document to collection
      evt.preventDefault()
      console.log('Adding new point')
      geoFirestore.add({ 
        coordinates: new firebase.firestore.GeoPoint(37.79, -122.41),
        info1:'string of info 1',
        info2:'string of info 2',
        info3:'string of info 3',
        obj1:{
          num1: 1,
          info4:'string of info 4'
        }
        }).then((docRef) => {
        console.log(docRef.id); // ID of newly added document
      }, (error) => {
        console.log('Error: ' + error);
      });
    },
    findPoints (evt) {
      evt.preventDefault()
      
      console.log('finding points...')
      const geoQuery = geoFirestore.query({
        center: new firebase.firestore.GeoPoint(this.clientPosition.lat, this.clientPosition.lng),
        radius: 200
        //query: (ref) => ref//.where(key<String>, '==', something<Any>)
      });
      console.log(`Center of query: ${JSON.stringify(geoQuery.center())}`)
      const query = geoQuery.query();  // A query object
      
      query.get().then(querySnapshot => {
        querySnapshot.forEach(doc => {
          //console.log(doc.data())
          this.locations.push(doc.data())
        })
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.location{
  background-color: rgb(233, 233, 233);
  margin:5px;
  padding: 5px;
  width: 700px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
