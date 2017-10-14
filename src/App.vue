<<template>
  <div class="box" style="margin-top:45px;">
    <p class="title">{{ movieId }}</p> 
    <p class="title" v-if="movieId === ''">กรุณาเลือกหนัง</p>
    <button v-if="movieId !== ''" class="button" @click="deleteAll(movieId)">Clear ที่นั่งทั้งหมด</button>
    <p class="title" v-if="movieId !== ''" style="padding:20px;">Count: {{status.count }} , Price: {{status.price}} </p>
    <Movie @chooseMovie="handleChooseMovie" :movieId="movieId" />
    
    <seat v-if="movieId !== ''"
    @chooseSeat="handleChooseSeat" 
    :movieId="movieId" 
    :selectSeats="selectSeats"
    :firebaseSeats="firebaseSeats"
    />
  </div>
  
  
  
  
</template>

<script>
  import firebase from 'firebase'
  import _ from 'lodash'


  import Movie from './Movie.vue'
  import Seat from './Seat.vue'
  import { pushToArray } from './lib.js'

  const config = {
      databaseURL: "https://movie-9cdc3.firebaseio.com",
  }

  firebase.initializeApp(config)

  const db = firebase.database()
  

  export default {
    components: { Movie , Seat },
    data() {
      return {
        movieId: '',
        selectSeats:[],
        firebaseSeats: [],
        status: {count: 0, price: 0},
        myQueue: []
      }
    },
    methods: {
      handleChooseMovie(movieId){
        if (this.status.count != 0){
          if (confirm('Data will be lost ?')){
            this.status = {count: 0 , price: 0}
            this.selectSeats = []
          }
          else {
            return false
          }
        }
        this.movieId = movieId

        const movieRef = db.ref('/').child(this.movieId)
        movieRef.on('value', snapshot => {
            
            const seats = snapshot.val()

            this.firebaseSeats = []

            _.forOwn(seats, s => {
                pushToArray(s, this.firebaseSeats)
            })
        })
      },
      handleChooseSeat(seat){
        const idsFirebase = this.firebaseSeats.map(s => s.id)
        const idxFirebase = idsFirebase.indexOf(seat.id)
        const ids = this.myQueue.map(s => s.id)
        const idx = ids.indexOf(seat.id)
        if (idxFirebase !== -1 && idx === -1){
        return alert('ที่นั่งนี้ถูกจองแล้ว!')
        }
     
      
        
        pushToArray(seat, this.selectSeats)
        pushToArray(seat, this.myQueue)

        const movieRef = db.ref('/').child(this.movieId)
        movieRef.push(seat)
        
        this.status = this.selectSeats.reduce((summary, s) => {
           summary.count++
           summary.price += s.price

           return summary
        }, { count:0,price:0})
      },
      deleteAll(movieId) {
        db.ref('/').child(this.movieId).remove()
        return alert('Clear ข้อมูลเรียบร้อย')
      }
    }
  }
</script>
<style>
</style>