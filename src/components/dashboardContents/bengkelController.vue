<template> 
    <v-container> 
        <v-card> 
            <v-container grid-list-md mb-0>
                <h2 class="text-md-center">Data Bengkel</h2> 
                <v-layout row wrap style="margin:10px"> 
                    <v-flex xs6> 
                        <v-btn depressed dark rounded style="text-transform: none !important;" color = "green accent-3" @click="dialog = true" > 
                            <v-icon size="18" class="mr-2">mdi-pencil-plus</v-icon> Tambah Bengkel 
                        </v-btn> 
                    </v-flex> 
                    <v-flex xs6 class="text-right"> 
                        <v-text-field v-model="keyword" append-icon="mdi-search" label="Search" hide-details ></v-text-field> 
                    </v-flex> 
                </v-layout> 
                <v-data-table :headers="headers" :items="bengkels" :search="keyword" :loading="load" > 
                    <template v-slot:body="{ items }"> 
                        <tbody> 
                            <tr v-for="(item,index) in items" :key="item.id"> 
                                <td>{{ index + 1 }}</td> 
                                <td>{{ item.name }}</td> 
                                <td>{{ item.address}}</td> 
                                <td>{{ item.phoneNumber }}</td> 
                                <td>{{ item.created_at }}</td>
                                <td class="text-center"> 
                                    <v-btn icon color="indigo" light @click="editHandler(item)" > 
                                        <v-icon>mdi-pencil</v-icon> 
                                    </v-btn> 
                                    <v-btn icon color="error" light @click="deleteData(item.id)" > 
                                        <v-icon>mdi-delete</v-icon> 
                                    </v-btn> 
                                </td> 
                            </tr> 
                        </tbody> 
                    </template> 
                </v-data-table> 
            </v-container> 
        </v-card> 
        <v-dialog v-model="dialog" persistent max-width="600px"> 
            <v-card> 
                <v-card-title> 
                    <span class="headline">Bengkel Profile</span> 
                </v-card-title> 
                <v-card-text> 
                    <v-container> 
                        <v-row> 
                            <v-col cols="12"> 
                                <v-text-field label="Name*" v-model="form.name" required></v-text-field> 
                            </v-col> 
                            <v-col cols="12"> 
                                <v-text-field label="address*" v-model="form.address" required></v-text-field> 
                            </v-col> 
                            <v-col cols="12"> 
                                <v-text-field label="phoneNumber*" v-model="form.phoneNumber" required></v-text-field> 
                            </v-col>
                        </v-row> 
                    </v-container> 
                    <small>*indicates required field</small> 
                </v-card-text> 
                <v-card-actions> 
                    <v-spacer></v-spacer> 
                    <v-btn color="blue darken-1" text @click="dialog = false">Close</v-btn> 
                    <v-btn color="blue darken-1" text @click="setForm()">Save</v-btn> 
                </v-card-actions> 
            </v-card> 
        </v-dialog> 
        <v-snackbar 
            v-model="snackbar"
            :color="color" 
            :multi-line="true" 
            :timeout="3000" 
        > {{ text }} 
            <v-btn dark text @click="snackbar = false" > Close </v-btn> 
        </v-snackbar> 
    </v-container> 
</template> 
<script> 
    export default { 
        data () { 
            return { 
                dialog: false, 
                keyword: '', 
                items: [],
                headers: [ 
                    { 
                        text: 'No',
                        value: 'no', 
                    }, 
                    { 
                        text: 'Name', 
                        value: 'name' 
                    }, 
                    { 
                        text: 'Address', 
                        value: 'address' 
                    }, 
                    { 
                        text: 'Phone Number', 
                        value: 'phoneNumber' 
                    }, 
                    { 
                        text: 'Created at', 
                        value: 'created_at' 
                    },
                    { 
                        text: 'Aksi', 
                        value: null 
                    }, 
                ], 
                bengkels: [], 
                snackbar: false, 
                color: null, 
                text: '', 
                load: false,
                form: { 
                    name : '', 
                    address : '', 
                    phoneNumber : '' 
                }, 
                bengkel : new FormData, 
                typeInput: 'new', 
                errors : '', 
                updatedId : '', 
            } 
        }, 
        methods:{ 
            getData(){ 
                var uri = this.$apiUrl + '/bengkel' 
                this.$http.get(uri).then(response =>{ 
                    this.bengkels=response.data.message 
                }) 
            }, 
            sendData(){ 
                this.bengkel.append('name', this.form.name); 
                this.bengkel.append('address', this.form.address); 
                this.bengkel.append('phoneNumber', this.form.phoneNumber);
                var uri =this.$apiUrl + '/bengkel' 
                this.load = true 
                this.$http.post(uri,this.bengkel).then(response =>{ 
                    this.snackbar = true; //mengaktifkan snackbar 
                    this.color = 'green'; //memberi warna snackbar 
                    this.text = response.data.message; //memasukkan pesan ke snackbar 
                    this.load = false; this.dialog = false 
                    this.getData(); //mengambil data user 
                    this.resetForm(); 
                }).catch(error =>{ 
                    this.errors = error 
                    this.snackbar = true; 
                    this.text = 'Try Again'; 
                    this.color = 'red'; 
                    this.load = false; 
                }) 
            }, 
            updateData(){ 
                this.bengkel.append('name', this.form.name); 
                this.bengkel.append('address', this.form.address); 
                this.bengkel.append('phoneNumber', this.form.phoneNumber); 
                var uri = this.$apiUrl + '/bengkel/' + this.updatedId; 
                this.load = true 
                this.$http.post(uri,this.bengkel).then(response =>{
                    this.snackbar = true; //mengaktifkan snackbar 
                    this.color = 'green'; //memberi warna snackbar 
                    this.text = response.data.message; //memasukkan pesan ke snackbar 
                    this.load = false; this.dialog = false 
                    this.getData(); //mengambil data user 
                    this.resetForm(); this.typeInput = 'new'; 
                }).catch(error =>{ 
                    this.errors = error 
                    this.snackbar = true; 
                    this.text = 'Try Again'; 
                    this.color = 'red'; 
                    this.load = false; 
                    this.typeInput = 'new'; 
                }) 
            }, 
            editHandler(item){ 
                this.typeInput = 'edit'; 
                this.dialog = true; 
                this.form.name = item.name; 
                this.form.address = item.address; 
                this.form.phoneNumber = item.phoneNumber, 
                this.updatedId = item.id 
            }, 
            deleteData(deleteId){ 
                //mengahapus data 
                var uri = this.$apiUrl + '/bengkel/' + deleteId; 
                //data dihapus berdasarkan id 
                this.$http.delete(uri).then(response =>{ 
                    this.snackbar = true; 
                    this.text = response.data.message; 
                    this.color = 'green' 
                    this.deleteDialog = false; 
                    this.getData(); 
                }).catch(error =>{ 
                    this.errors = error 
                    this.snackbar = true; 
                    this.text = 'Try Again'; 
                    this.color = 'red'; 
                }) 
            }, 
            setForm(){ 
                if (this.typeInput === 'new') { 
                    this.sendData() 
                } else { 
                    console.log("dddd") 
                    this.updateData() 
                } 
            }, 
            resetForm(){ 
                this.form = {
                    name : '', 
                    address : '', 
                    phoneNumber : '' 
                } 
            } 
        }, 
        mounted(){ 
            this.getData(); 
        }, 
    } 
</script>