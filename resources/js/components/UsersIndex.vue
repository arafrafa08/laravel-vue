<template>
    <div class="users">
        <div class="btn-toolbar mb-3" role="toolbar" aria-label="Toolbar with button groups">
            <router-link class="btn btn-secondary"  :to="{ name: 'users.create' }">Create</router-link>
        </div>
        <div v-if="error" class="error">
            <p>{{ error }}</p>
        </div>

        <div v-if="users">

        <table class="table table-striped table-hover">
            <thead class="thead-light">
            <tr>
                <th scope="col">Name</th>
                <th scope="col">Email</th>
                <th scope="col">Action</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="{ id, name, email } in users">
                <td>{{name}}</td>
                <td>{{email}}</td>
                <td>
                    <div class="btn-toolbar mb-3" role="toolbar" aria-label="Toolbar with button groups">
                        <div class="btn-group mr-2" role="group" aria-label="First group">
                            <router-link  class="btn btn-secondary" :to="{ name: 'users.edit', params: { id } }">Edit</router-link>
                            <button type="button" class="btn btn-danger" data-toggle="modal" data-target="#exampleModal" @click="showModal({id, name, email})">Delete</button>
                        </div>
                    </div>
                </td>
            </tr>
            </tbody>
        </table>
        </div>

        <div class="btn-toolbar mb-3" role="toolbar" aria-label="Toolbar with button groups">
            <div class="btn-group mr-2" role="group" aria-label="First group">
                <button type="button" class="btn btn-secondary" :disabled="! prevPage" @click.prevent="goToPrev">Previous</button>
                <button type="button" class="btn btn-secondary" disabled>{{ paginationCount }}</button>
                <button type="button" class="btn btn-secondary" :disabled="! nextPage" @click.prevent="goToNext">Next</button>
            </div>
        </div>
        <modal v-show="isModalVisible" :user="selectedUser" @close="closeModal" :method="deleteUser"></modal>
    </div>
</template>
<script>
    import axios from 'axios';
    import api from '../api/users';
    const getUsers = (page, callback) => {
        const params = { page };
        axios
            .get('/api/users', { params })
            .then(response => {
                callback(null, response.data);
            }).catch(error => {
            callback(error, error.response.data);
        });
    };
    export default {
        data() {
            return {
                renderComponent: true,
                isModalVisible: false,
                users: null,
                selected: null,
                meta: {
                    current_page: null,
                    last_page: null,
                    to: null,
                    total: null,
                    from: null
                },
                links: {
                    first: null,
                    last: null,
                    next: null,
                    prev: null,
                },
                error: null,
            };
        },
        computed: {
            nextPage() {
                if (! this.meta || this.meta.current_page === this.meta.last_page) {
                    return;
                }
                return this.meta.current_page + 1;
            },
            prevPage() {
                if (! this.meta || this.meta.current_page === 1) {
                    return;
                }
                return this.meta.current_page - 1;
            },
            currentPage() {
                return this.meta.current_page;
            },
            paginationCount() {
                if (! this.meta) {
                    return;
                }
                const { current_page, last_page } = this.meta;
                return `${current_page} of ${last_page}`;
            },
            selectedUser(){
                return this.selected;
            }
        },
        beforeRouteEnter (to, from, next) {
            getUsers(to.query.page, (err, data) => {
                next(vm => vm.setData(err, data));
            });
        },
        // when route changes and this component is already rendered,
        // the logic will be slightly different.
        beforeRouteUpdate (to, from, next) {
            this.users = this.links = this.meta = null;
            getUsers(to.query.page, (err, data) => {
                this.setData(err, data);
                next();
            });
        },
        updated(){
            vm.$forceUpdate();
        },
        methods: {
            showModal(data) {
                this.selected = data;
                this.isModalVisible = true;

            },
            closeModal() {
                this.isModalVisible = false;
            },
            goToNext() {
                this.$router.push({
                    query: {
                        page: this.nextPage,
                    },
                });
            },
            goToPrev() {
                this.$router.push({
                    name: 'users.index',
                    query: {
                        page: this.prevPage,
                    }
                });
            },
            goHome() {
                this.$router.push({
                    name: 'users.index',
                    query: {
                        page: 1,
                    }
                });
            },
            setData(err, data) {
                if (err) {
                    this.error = err.toString();
                } else {
                    this.users = data.data;
                    this.links = {
                        first: data.first_page_url,
                        next: data.next_page_url,
                        prev: data.prev_page_url,
                        last: data.last_page_url
                    };
                    this.meta = {
                        current_page: data.current_page,
                        last_page: data.last_page,
                        to: data.to_page,
                        total: data.total,
                        from: data.from
                    };
                }
            },
            deleteUser(id) {
                api.delete(id).then((response) => {
                    if(response) {
                        this.selected = null;
                        setTimeout(() => this.$router.push({ name: 'users.index',query: {
                                page: 1,
                            } }), 500);
                    }
                });
            }
        }
    }
</script>