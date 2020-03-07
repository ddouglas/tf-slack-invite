<template>
    <div>Loading...</div>
</template>

<script>
import queryString from "query-string";
import axios from "axios";
import router from "../router";

export default {
    name: "callback",
    async mounted() {
        const parsed = queryString.parse(window.location.search);
        const response = await axios.post(
            "http://homeserver:5000/slack/invite",
            {
                code: parsed.code,
                state: parsed.state
            }
        );

        if (response.status == 200) {
            localStorage.setItem("token", response.data.access_token);
        }

        router.push({ path: "/" });
    }
};
</script> 