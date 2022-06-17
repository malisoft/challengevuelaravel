<script setup>
import { Head, Link } from "@inertiajs/inertia-vue3";

defineProps({
    canLogin: Boolean,
    canRegister: Boolean,
    laravelVersion: String,
    phpVersion: String,
});
</script>

<template>
    <Head title="Welcome to this challenge" />

    <div
        class="relative flex items-top justify-center min-h-screen bg-gray-100 dark:bg-gray-900 sm:items-center sm:pt-0"
    >
        <div
            v-if="canLogin"
            class="hidden fixed top-0 right-0 px-6 py-4 sm:block"
        >
            <Link
                v-if="$page.props.user"
                :href="route('dashboard')"
                class="text-sm text-gray-700 underline"
            >
                Dashboard
            </Link>

            <template v-else>
                <Link
                    :href="route('login')"
                    class="text-sm text-gray-700 underline"
                >
                    Log in
                </Link>

                <Link
                    v-if="canRegister"
                    :href="route('register')"
                    class="ml-4 text-sm text-gray-700 underline"
                >
                    Register
                </Link>
            </template>
        </div>

        <form v-if="!token" @submit.prevent="submit">
            <button type="submit">Submit</button>
            <div>
                <span>{{ message }}</span>
            </div>
        </form>
	<div v-else>
		{{countries}}
	</div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            grant_type: "password",
            client_id: 1,
            client_secret: "AWAOhGzmnfx4FdyfIAjWwaMGBY6SUGKYC85L7vpm",
            username: "admin@shipedge.com",
            password: "Admin123",

            message: "",
            token: null,
	    countries:[],
        };
    },
    created() {
        if (localStorage.getItem("client_credentials")) {
            this.token = localStorage.getItem("client_credentials");
	    this.getCountries();
        }
	
    },

    methods: {
	getCountries(){
            fetch("https://condordemo.shipedge.com/api/1.0/countries", {
                method: "GET",
                header: {
                    Authorization:
                        "Bearer " + this.token,
                },
            })
                .then((response) => {
                    return response.json();
                })
                .then((data) => {
                    console.log(data);
		    this.countries=data;
                });

	},
        submit() {
            this.message = "";
            fetch("https://condordemo.shipedge.com/oauth/token", {
                method: "POST",
                body: JSON.stringify({
                    client_id: this.client_id,
                    client_secret: this.client_secret,
                    grant_type: this.grant_type,
                    username: this.username,
                    password: this.password,
                }),
            })
                .then((response) => {
                    return response.json();
                })
                .then((data) => {
                    if (data.access_token) {
                        localStorage.setItem(
                            "client_credentials",
                            data.access_token
                        );

                        this.token = data.access_token;
			this.getCountries();
                    } else {
                        this.message = "Error al traer las credenciales";
                    }
                });
        },
    },
};

</script>
