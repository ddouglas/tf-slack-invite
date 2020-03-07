<template>
    <div class="mt-2">
        <b-row v-if="invite.done">
            <b-col lg="12">
                <b-alert v-if="invite.success" variant="success" show>
                    <h4>Congratulations, You're almost there!</h4>
                    {{invite.message}}
                </b-alert>
                <b-alert variant="danger" v-else show>
                    <h4>Uh Oh! We encountered an error</h4>
                    {{invite.message}}
                    <br />
                    {{invite.error}}
                </b-alert>
            </b-col>
        </b-row>
        <b-row v-if="invite.message != ''">
            <b-row v-if="invite.success"></b-row>
            <b-row v-if="!invite.success"></b-row>
        </b-row>
        <b-row>
            <b-col lg="6">
                <b-card header="Welcome to the Tweetfleet Slack Inviter">
                    <b-card-text>
                        Before getting started, please make sure you have an email address that you are comfortable with the administartors of the TF Slack having access to. We
                        will never use this email address outside of this inviter, but slack requires a valid email address for the invitation. Once you are ready, click the "Login With EveOnline SSO" below to kick off the invitation process.
                        <a
                            :href="authURL"
                        >
                            <img
                                src="https://web.ccpgamescdn.com/eveonlineassets/developers/eve-sso-login-black-large.png"
                                class="mx-auto d-block mt-1"
                            />
                        </a>
                    </b-card-text>
                </b-card>
            </b-col>
            <b-col lg="6" v-if="character && token">
                <b-card :header="'Welcome Back ' + character ">
                    <b-card-text>
                        The final step to this process is to input an email address below. From there we will send you an invitation, via Slack, to the TF Slack and you can join us there
                        <b-form @submit="onSubmit" @reset="onReset" class="mt-2">
                            <b-form-group label="My Email Address:" label-for="email-input">
                                <b-form-input id="email-input" v-model="email" />
                            </b-form-group>
                            <b-button type="submit" variant="primary">Submit</b-button>
                            <b-button type="reset" variant="danger" class="ml-1">Reset</b-button>
                        </b-form>
                    </b-card-text>
                </b-card>
            </b-col>
        </b-row>
    </div>
</template>

<script>
import axios from "axios";
import jwt_decode from "jwt-decode";
import queryString from "query-string";
import router from "../router";

export default {
    name: "Home",
    data() {
        return {
            email: "",
            authURL: "",
            token: localStorage.getItem("token") || "",
            character: "",
            invite: {
                done: false,
                success: false,
                error: "",
                message: ""
            }
        };
    },
    methods: {
        async onSubmit(evt) {
            evt.preventDefault();
            await axios
                .post(
                    "http://homeserver:5000/slack/invite/send",
                    {
                        email: this.email
                    },
                    {
                        headers: {
                            Authorization: "Bearer " + this.token
                        }
                    }
                )
                .then(response => {
                    this.invite.done = true;
                    this.invite.success = true;
                    this.invite.message =
                        "Congrats " +
                        this.character +
                        ". You have successfully been invited to Tweetfleet Slack. You should receive an email at " +
                        this.email +
                        " from Slack with the invitation link to follow to complete your registeration.";
                })
                .catch(error => {
                    let response = error.response;
                    if (response.data.message) {
                        this.invite.error = response.data.message;
                    }
                    if (response.data.error) {
                        this.invite.error = response.data.error;
                    }
                    this.invite.done = true;
                    this.invite.success = false;
                    this.invite.message =
                        "Our apologies " +
                        this.character +
                        ". We're unable to invite you to slack due to an error we received from Slack. The error is below for your information";

                    localStorage.removeItem("token");
                    this.token = "";
                });
        },
        onReset() {
            localStorage.removeItem("token");
            window.location = "/";
        }
    },
    async mounted() {
        if (window.location.search.length > 1) {
            const parsed = queryString.parse(window.location.search);
            let response;
            try {
                response = await axios.post(
                    "http://homeserver:5000/slack/invite",
                    {
                        code: parsed.code,
                        state: parsed.state
                    }
                );
            } catch {
                window.location = "/";
            }

            localStorage.setItem("token", response.data.access_token);
            this.token = response.data.access_token;
        }
        let { data } = await axios.get("http://homeserver:5000/slack/invite");
        this.authURL = data.url;

        if (this.token) {
            this.character = jwt_decode(this.token).name;
        }
    }
};
</script> 
 