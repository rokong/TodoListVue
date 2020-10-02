<template>
    <div id="container">
        <div class="page-header">
            <h1 class="text-center">Contact Manage Application</h1>
            <p>Dynamic Component + EventBus + Axios</p>
        </div>
        <component :is="currentView" :contact="contact"></component>
        <contactList :contactlist="contactlist"></contactList>
    </div>
</template>

<script>
import ContactList from './components/ContactList';
import AddContact from './components/AddContact';
import UpdateContact from './components/UpdateContact';
import UpdatePhoto from './components/UpdatePhoto';
import CONF from './Config.js';
import eventBus from './EventBus.js';

export default {
    name : 'app',
    components : { ContactList, AddContact, UpdateContact, UpdatePhoto },
    data : function(){
        return {
            currentView : null,
            contact : { no:0, name:'', tel:'', address:'', photo:'' },
            contactlist : {
                pageno:1, pagesize: CONF.PAGESIZE, totalcount:0, contacts:[]
            }
        }
    },
    mounted: function(){
        this.fetchContacts();
        eventBus.$on('pageChanged', (page) => {
            this.pageChanged(page);
        });
        eventBus.$on("addContactForm", () => {
            this.currentView = 'addContact';
        });
        eventBus.$on("cancel", () => {
            this.currentView = null;
        });
        eventBus.$on("editContactForm", (no) => {
            this.fetchContactOne(no)
            this.currentView = 'updateContact';
        });
        eventBus.$on("editPhoto", (no) => {
            this.fetchContactOne(no)
            this.currentView = 'updatePhoto';
        });
    },
    methods: {
        pageChanged : function(page){
            this.contactlist.pageno = page;
            this.fetchContacts();
        },
        fetchContacts : function(){
            this.$axios.get(CONF.FETCH, {
                params : {
                    pageno:this.contactlist.pageno,
                    pagesize:this.contactlist.pagesize
                }
            }).then((response) => {
                this.contactlist = response.data;
            }).catch((ex) => {
                console.log('ERROR occured in fetchContacts()', ex);
                this.contactlist.contacts = [];
            })
        },
        fetchContactOne : function(no) {
            this.$axios.get(CONF.FETCH_ONE.replace("${no}", no))
            .then((response) => {
                this.contact = response.data;
            })
            .catch((ex)=> {
                console.log('fetchContactOne failed', ex);
            })
        },
    }
}
</script>

<style scoped>
#container {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>