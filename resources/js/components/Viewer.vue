<style>
    .bi:hover{
        cursor: pointer;
        color:hsl(240,65%,75%);
    }

    .active-cyan input[type=text] {
        border-bottom: 1px solid #4dd0e1;
        box-shadow: 0 1px 0 0 #4dd0e1;
    }


    .clearfix{
        clear: both;}
</style>

<template>
    <div style="float: left; width:100%;">

        <!--Modal -->
        <modal></modal>






        <div style="height: 60px; width: 100%;">

            <div style="width: 100px; height: 50px; float: left;">
                <nav aria-label="Page navigation example">
                    <div style=" width: 250px; display: inline-block;">
                        <ul class="pagination" style="width: 250px;">
                            <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#"
                                                                                                            @click="fetchMedia(pagination.prev_page_url)">Previous</a></li>
                            <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page}} of {{ pagination.last_page }}</a></li>
                            <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#"
                                                                                                            @click="fetchMedia(pagination.next_page_url)">Next</a></li>
                        </ul>
                    </div>
                </nav>
            </div>

            <div style="width: 300px; height: 50px; float: right" class="mr-100">
                <!-- Search form -->
                <form @submit.prevent="" class="form-inline d-flex justify-content-center md-form form-sm active-cyan active-cyan-2 mt-2" style="width: 300px;">
                    <svg class="bi bi-search" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg"
                         @click="fetchMedia()">
                        <path fill-rule="evenodd" d="M10.442 10.442a1 1 0 011.415 0l3.85 3.85a1 1 0 01-1.414 1.415l-3.85-3.85a1 1 0 010-1.415z" clip-rule="evenodd"/>
                        <path fill-rule="evenodd" d="M6.5 12a5.5 5.5 0 100-11 5.5 5.5 0 000 11zM13 6.5a6.5 6.5 0 11-13 0 6.5 6.5 0 0113 0z" clip-rule="evenodd"/>
                    </svg>
                    <input class="form-control form-control-sm ml-3 w-75" type="text" placeholder="Search"
                           aria-label="Search" v-model="meta_search.search_key" v-on:keyup.enter="fetchMedia()">
                </form>
            </div>
        </div>

<div class="clearfix"></div>


        <div class="shadow-lg p-3 bg-white rounded-lg card card-body mb-2 mr-2 float-left" style="max-height: 362px; height: 362px; width:240px; max-width: 240px" v-for="(photo, index) in media"
             v-bind:key="photo.id">
            <h4 style="text-align: center">{{ photo.name }}</h4>
            <img @click="openModal(index)" v-bind:src="photo.path" style="cursor: pointer; max-height: 188px; max-width: 200px; height: 188px; width: 200px;" />
            <p class="mt-1" style="height: 75px; max-height:75px; overflow-y: auto">{{ photo.description}} </p>
            <div>
                <ul style="list-style: none; padding: 0px;">
                    <li><span title="favorite"><svg @click="setMediaFav(photo.id)" class="bi bi-star"  width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" d="M2.866 14.85c-.078.444.36.791.746.593l4.39-2.256 4.389 2.256c.386.198.824-.149.746-.592l-.83-4.73 3.523-3.356c.329-.314.158-.888-.283-.95l-4.898-.696L8.465.792a.513.513 0 00-.927 0L5.354 5.12l-4.898.696c-.441.062-.612.636-.283.95l3.523 3.356-.83 4.73zm4.905-2.767l-3.686 1.894.694-3.957a.565.565 0 00-.163-.505L1.71 6.745l4.052-.576a.525.525 0 00.393-.288l1.847-3.658 1.846 3.658a.525.525 0 00.393.288l4.052.575-2.906 2.77a.564.564 0 00-.163.506l.694 3.957-3.686-1.894a.503.503 0 00-.461 0z" clip-rule="evenodd"/>
                    </svg></span></li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script>
    var mediaG = [],
        media_indexG;

    export default {
        data() {
            return {
                media: [],
                photo: {
                    id: '',
                    path: '',
                    name: '',
                    description: '',
                },
                meta_search: {
                    search_key: ''
                },
                pagination: {},
                modalOpenStatus: false,
                media_index: 0,
            }
        },
        created(){
            this.fetchMedia();


        },

        methods: {
            fetchMedia(page_url){
                let vm = this;
                page_url = page_url || 'api/get-media'
                fetch(page_url,{
                    method: 'post',
                    body: JSON.stringify(this.meta_search),
                    headers:{
                        'Accept': 'application/json',
                        'Content-Type': 'application/json',
                        'Authorization': 'Bearer ' + window.localStorage.getItem('token'),
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.media = data['objList'].data;
                        mediaG = this.media;
                        vm.makePagination(data['objList']);
                    })
                   .catch(err => console.log(err));
        },
            makePagination(meta){
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: meta.next_page_url,
                    prev_page_url: meta.prev_page_url
                }

                this.pagination = pagination;
            },
         setMediaFav(media_id) {
                if(window.localStorage.getItem('token') == null) {
                    this.$router.push('login');
                    return;
                }

             fetch('api/set-as-fav',{
                 method: 'post',
                 body: JSON.stringify({
                     'media_id': media_id
                 }),
                 headers:{
                     'Accept': 'application/json',
                     'Content-Type': 'application/json',
                     'Authorization': 'Bearer ' + window.localStorage.getItem('token'),
                 }
             })
                 .then(res => res.json())
                 .then(data => {
                     if(data.status == 0)
                         alert(data.message);
                     else
                      alert('Media set as favorite');
                 })
                 .catch(err => console.log(err));
         },

         openModal(index) {
             $('#btn-modal').click();
             this.media_index = index;
             media_indexG = index;

             $('#modal-media').attr('src', this.media[index].path);
         },
         media_next(){

                alert('dd');
                if(this.media_index < this.media_index.length)
                  this.media_index ++;
             $('#modal-media').attr('src', this.media[this.media_index].path);
         },

         media_prev(){
             if(this.media_index > 0)
                 this.media_index--;
             $('#modal-media').attr('src', this.media[this.media_index].path);
         },


        }
    };


    $(document).ready(function () {
        $('#btn-next').click(function () {
            if(media_indexG < mediaG.length)
                media_indexG ++;
            $('#modal-media').attr('src', mediaG[media_indexG].path);
        })

        $('#btn-prev').click(function () {
            if(media_indexG > 0)
                media_indexG--;
            $('#modal-media').attr('src', mediaG[media_indexG].path);
        })
    })


</script>