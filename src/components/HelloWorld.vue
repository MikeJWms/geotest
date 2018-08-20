<template>
  <div class="hello">
    <button @click="geoTest">Launch Script</button>
  </div>
</template>

<script>
import firebase from 'firebase'
import 'firebase/firestore' //if use firestore
import GeoFirestore from 'geofirestore'

if (!firebase.apps.length) {
  firebase.initializeApp({
    apiKey: "xxxx",
    authDomain: "xxxx",
    databaseURL: "xxxx",
    projectId: "xxxx",
    storageBucket: "xxxx",
    messagingSenderId: "xxxx"
  })
}

firebase.firestore().settings({ timestampsInSnapshots: true })
const db = firebase.firestore()

// Create a Firebase reference where GeoFirestore will store its information
const collectionRef = firebase.firestore().collection('geotest');

// Create a GeoFirestore index
const geoFirestore = new GeoFirestore(collectionRef);

export default {

  name: 'HelloWorld',
  props: {
    msg: String
  },
  methods: {
    geotest (evt) {
      evt.preventDefault()
      geoFirestore.add({ coordinates: new firebase.firestore.GeoPoint(37.79, -122.41)}).then((docRef) => {
        console.log(docRef.id); // ID of newly added document
      }, (error) => {
        console.log('Error: ' + error);
      });
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
