<template>
  <v-container fluid>
    <v-slide-y-transition mode="out-in">
      <v-layout column align-center>
        <v-flex xs12 sm6>
          <v-autocomplete
            @select='loadOwner'
            :items="owners"
            v-model="owner_id"
            item-text="text"
            item-value="value"
            label="Select an owner"
          ></v-autocomplete>
      </v-flex>

        <v-data-table
          :headers="headers"
          :items="items"
          :loading="loading"
          hide-actions
          :pagination.sync="pagination"
          class="elevation-1"
        >
          <template slot="items" slot-scope="props">
            <td class="text-xs-left" v-bind:class="{ eliminated: props.item.eliminated }">#{{ props.item.jersey }} {{ props.item.full_name }}</td>
            <td class="text-xs-left hidden-xs-only" v-bind:class="{ eliminated: props.item.eliminated }">{{ props.item.school }}</td>
            <td class="text-xs-left hidden-xs-only" v-bind:class="{ eliminated: props.item.eliminated }">{{ props.item.seed }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.scoring_average }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.round1 }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.round2 }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.round3 }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.round4 }}</td>
            <td class="text-xs-center hidden-xs-only">{{ props.item.round5 }}</td>
            <td class="text-xs-center">{{ props.item.round6 }}</td>
            <td class="text-xs-center">{{ props.item.total }}</td>
           </template>
        </v-data-table>

      </v-layout>
    </v-slide-y-transition>
  </v-container>
</template>

<script>
  var defaultHeaders = {
    Accept: 'application/json',
    'Content-Type': 'application/json',
    'Cache-Control': 'no-cache',
  };

  var defaultOptions = {
    method: 'GET',
    headers: defaultHeaders,
    mode: 'cors',
  };
  export default {
    data () {
      return {
        owner_id: 1,
        owners: [],
        section_title: "",
        mobile: true,
        loading: true,
        pagination: {'sortBy': 'total', 'descending': true, 'rowsPerPage': -1},
        draft_set: false,
        headers: [
          { text: 'Player', value: 'full_name' },
          { text: 'School', value: 'school', class: 'hidden-xs-only'},
          { text: 'Seed', value: 'seed', class: 'hidden-xs-only'},
          { text: 'Scoring Average', value: 'scoring_average', class: 'hidden-xs-only'},
          { text: 'Round 1', value: 'round1', class: 'hidden-xs-only'},
          { text: 'Round 2', value: 'round2', class: 'hidden-xs-only'},
          { text: 'Sweet Sixteen', value: 'round3', class: 'hidden-xs-only'},
          { text: 'Elite Eight', value: 'round4', class: 'hidden-xs-only'},
          { text: 'Final Four', value: 'round5', class: 'hidden-xs-only'},
          { text: 'Championship', value: 'round6'},
          { text: 'Total', value: 'total'}
        ],
        items: []
      }
    },
    created: function () {},
    mounted: function () {
      this.owner_id = parseInt(this.$route.params.id) || 1;
      if(window.innerWidth < 1100){
        this.mobile = true;
      } else {
        this.mobile = false;
      }
      this.$nextTick(function() {
        window.addEventListener('resize', function() {
          if(window.innerWidth < 1100){
            this.mobile = true;
          } else {
            this.mobile = false;
          }
        });
      })
      this.getOwners()
      this.interval1 = setInterval(function(){
        this.loadOwner()
      }.bind(this), 180000);
      this.loadOwner()
    },
    methods: {
      getOwners(){
        fetch(`${process.env.VUE_APP_BACKEND_URL}/owners`, defaultOptions)
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            data.forEach((element) => {
              var entry = {
                value: element.id,
                text: `${element.display_name} (${element.name})`
              }
              this.owners.push(entry)
            })
          })
      },
      loadOwner(){
        fetch(`${process.env.VUE_APP_BACKEND_URL}/everything/owner/${this.owner_id}`, defaultOptions)
          .then((response) => {
            return response.json();
          })
          .then((data) => {
            this.items = [];
            if (data.length === 0) {
              this.loading = false;
              return;
            }
            this.section_title = `${data[0].display_name} (${data[0].name})`;
            data.forEach((element) => {
              var eliminated_player = false;
              if (element.eliminated > 0 ){
                eliminated_player = true;
              }
              var entry = {
                value: false,
                full_name: element.full_name,
                espn_id: element.espn_id,
                scoring_average: element.scoring_average,
                school: `${element.school} ${element.mascot}`,
                team_id: element.team_id,
                seed: element.seed,
                region: element.region,
                jersey: element.jersey,
                eliminated: eliminated_player,
                round1: element.round1,
                round2: element.round2,
                round3: element.round3,
                round4: element.round4,
                round5: element.round5,
                round6: element.round6,
                total: element.total
              }
              this.items.push(entry);
              this.loading = false;
            })
          })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
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
.eliminated {
  background-color: IndianRed !important;
}

</style>
