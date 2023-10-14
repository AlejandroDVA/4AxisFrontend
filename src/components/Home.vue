<template>
 
<div>
<div class="header">
    <h1 class="title">UF to CLP</h1> 
    <h2>Welcome back {{ userRole.charAt(0).toUpperCase() + userRole.slice(1) }}</h2>
</div>

<div class="input-columns">
    <div class="column">
        <p>Enter UF value:</p>
        <input type="text" v-model="value" placeholder="1.75" />
    </div>
    <div class="column">
      <p>Select date:</p>
      <input type="date" v-model="date" min="2018-01-01" max="2023-10-16" />
    </div>
    <div class="column">
        <br>
        <button v-on:click="find">Convert</button>
    </div>
    <div class="column">
        <br>
        <button v-on:click="logout">Logout</button>
    </div>
  </div>


</div><br>
<div>
    <p v-if="loading" role="alert">working on your petition...</p>
</div><br>
<div class="input-columns">
<div class="column">
    <h3>UF unit value: {{ result.valor || 0 }} CLP</h3>
</div>
<div class="column">
    <h3>CLP total Value: {{ result.responseCLP || 0}} CLP</h3>
</div>
</div><br>
<div class="input-columns">
    <div class="column">
        <button v-on:click="findHistory" v-if="userRole === 'admin'">Conversion History</button>
    </div>
    <div class="column" v-if="userRole === 'admin'" >
        <button @click="exportToExcel">Generate excel</button>
    </div>
</div><br>
<div v-if="userRole === 'admin'">
    <table v-if="showTable">
        <thead>
            <tr>
            <th>User</th>
            <th>Activity Date</th>
            <th>Origin Amount</th>
            <th>Conversion Date</th>
            <th>Coin Value</th>
            <th>Conversion Amount</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(item, index) in items" :key="index">
            <td>{{ item.user }}</td>
            <td>{{ item.activityDate }}</td>
            <td>{{ item.originAmount }}</td>
            <td>{{ item.convertionDate }}</td>
            <td>{{ item.coinValue }}</td>
            <td>{{ item.convertionAmount }}</td>
            </tr>
        </tbody>
    </table>
</div>
</template>


<script>
import axios from 'axios';
import * as XLSX from 'xlsx';
export default {
    name:'Home',
    data(){
        return {
            date: '',
            value: '',
            result: 'resultado',
            loading: null,
            userRole: '',
            history: '',
            userName: '',
            items: '',
            showTable: false
        }
        
    },
    methods:{
         async find(){
            this.loading = true;
            console.log("find",this.date,this.value);
            let result = await axios.post("https://4axisbackend.up.railway.app/convert",{
                date:this.formattedDate(),
                value: this.value,
                user: this.userName
            });
            if(result.status==200){
                console.log(result);
                //alert("UF converted Successfull");
                this.result = result.data;
                this.loading = null;
            }else{
                console.log(result);
                alert("invalid data");
                this.loading = null;
            }
        },
        async findHistory(){
            console.log("history")
            this.loading = true
            this.showTable = !this.showTable;
            console.log("history",this.date,this.value)
            let result = await axios.get("https://4axisbackend.up.railway.app/history");
            if(result.status==200){
                this.result=result.data;
                console.log(result);
                //alert("login success")
                this.loading = null;
                this.history= result.data;
                this.items = result.data;
            }else{
                console.log(result);
                alert("invalid credentials")
                this.loading = null
            }
        },
        exportToExcel() {
        const ws = XLSX.utils.json_to_sheet(this.items);
        const wb = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(wb, ws, 'Datos');
        XLSX.writeFile(wb, 'datos.xlsx');
        },
        formattedDate() {
        if (this.date) {
            const [year, month, day] = this.date.split('-');
            return `${day}-${month}-${year}`;
        }
        return '';
        },
        logout(){
            console.log("logout")
            localStorage.clear();
            this.$router.push({name:'SignUp'})
        }
    },
    mounted(){
        let user = localStorage.getItem('user-info');
        console.log(user)
        if(user){
            let userData = JSON.parse(user);
            this.userRole = userData[0].rol;
            this.userName = userData[0].name;
        }
        if(!user){
            this.$router.push({name:'SignUp'})
        }
        }
}
</script>

<style>
.input-columns {
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.column {
  flex: 1;
  margin: 10px;
}
</style>