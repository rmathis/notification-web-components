<template>
    <div class="notification-banner">
        <b-alert v-for="(notification, index) in notifications" :key="index" varient="info" show>
            <h4>
                <font-awesome-icon icon="info" />
                {{ notification.title }}
            </h4>
            <span v-html="notification.body" />
        </b-alert>
    </div>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faInfo } from '@fortawesome/free-solid-svg-icons/faInfo';
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import bAlert from 'bootstrap-vue/es/components/alert/alert';
import oidc from '@uportal/open-id-connect';
import { get } from 'axios';

library.add(faInfo);

export default {
    name: 'NotificationBanner',

    props: {
        debug: {
            type: Boolean,
            default: false
        },
        userInfoApiUrl: {
            type: String,
            default: '/uPortal/api/v5-1/userinfo'
        },
        notificationApiUrl: {
            type: String,
            default: '/NotificationPortlet/api/v2/notifications'
        },
        filter: {
            type: String,
            default: ''
        }
    },

    components: {
        'b-alert': bAlert,
        'font-awesome-icon': FontAwesomeIcon
    },

    data() {
        return {
            // list of notifications to display
            notifications: []
        };
    },

    methods: {
        async fetchNotifications() {
            // read props
            const { debug, notificationApiUrl, filter, userInfoApiUrl } = this;

            try {
                // Obtain an OIDC Id Token, except in debug mode
                const { encoded: token } = debug
                    ? { encoded: null }
                    : await oidc({ userInfoApiUrl });

                // gather notifications
                const querystring = filter ? '?' + filter : '';
                const { data: notifications } = await get(notificationApiUrl + querystring, {
                    withCredentials: true,
                    headers: {
                        Authorization: `Bearer ${token}`,
                        'content-type': 'application/jwt'
                    }
                });

                // store notifications to state
                this.notifications = notifications;
            } catch (err) {
                // eslint-disable-next-line no-console
                console.error(err);
            }
        }
    },

    // entrypoint
    created() {
        return this.fetchNotifications();
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.notification-banner {
    // core bootstrap framework
    @import '../../node_modules/bootstrap/scss/functions.scss';
    @import '../../node_modules/bootstrap/scss/variables.scss';
    @import '../../node_modules/bootstrap/scss/mixins.scss';
    // bootstrap styles needed by page
    @import '../../node_modules/bootstrap/scss/alert.scss';

    // custom styles
    margin: 1rem;
    text-align: left;
}

.notification-banner svg {
    height: 2rem;
    position: relative;
    top: 0.1rem;
}
</style>
