<script>
    import Multiselect from "vue-multiselect";

    export default {
        components: {
            "multiselect": Multiselect,
        },

        data: function () {
            return {
                accessToken: "",
                accessTokens: [],
                authorizedTokens: [],
                clientErrors: {},
                clientId: null,
                clientName: "",
                clients: [],
                redirectUrl: "",
                scopes: [],
                selectedScopes: [],
                showCreateClientButton: true,
                showUpdateClientButton: false,
                tokenErrors:{},
                tokenName: "",
            };
        },

        mounted() {
            this.getAuthorizedTokens();
            this.getClients();
            this.getAccessTokens();
            this.getScopes();
        },

        methods: {
            clearAccessToken: function () {
                this.accessToken = "";
            },

            clientFieldHasError: function (field) {
                return (((this.clientErrors || {})[field] || []).join(" ").length > 0);
            },

            clientFieldError: function (field) {
                return ((this.clientErrors || {})[field] || []).join(" ");
            },

            tokenFieldHasError: function (field) {
                return (((this.clientErrors || {})[field] || []).join(" ").length > 0);
            },

            tokenFieldError: function (field) {
                return ((this.clientErrors || {})[field] || []).join(" ");
            },

            getAuthorizedTokens: function () {
              Nova.request().get('/oauth/tokens')
                    .then(response => {
                        this.authorizedTokens = Object.assign([], response.data);
                    });
            },

            getAccessTokens: function () {
                Nova.request().get('/oauth/personal-access-tokens')
                    .then(response => {
                        this.accessTokens = Object.assign([], response.data);
                    });
            },

            getScopes() {
                Nova.request().get('/oauth/scopes')
                    .then(response => {
                        this.scopes = Object.assign([], response.data);
                    });
            },

            revokeAuthorizedToken: function (token) {
                Nova.request().delete('/oauth/tokens/' + token.id)
                    .then(response => {
                        this.getAuthorizedTokens();
                    });
            },

            getClients: function () {
                Nova.request().get('/oauth/clients')
                    .then(response => {
                        this.clients = Object.assign([], response.data);
                    });
            },

            storeClient: function () {
                this.persistClient(
                    'post',
                    '/oauth/clients'
                );
            },

            editClient: function (client) {
                this.clientId = client.id;
                this.clientName = client.name;
                this.redirectUrl = client.redirect;
                this.showUpdateClientButton = true;
                this.showCreateClientButton = false;
            },

            updateClient: function () {
                this.persistClient(
                    'put',
                    '/oauth/clients/' + this.clientId
                );

                this.showUpdateClientButton = false;
                this.showCreateClientButton = true;
            },

            persistClient(method, uri) {
                this.clientErrors = Object.assign({});

                Nova.request()[method](
                        uri,
                        {
                            name: this.clientName,
                            redirect: this.redirectUrl,
                        }
                    )
                    .then(response => {
                        this.getClients();
                        this.clientName = "";
                        this.redirectUrl = "";
                        this.clientErrors = Object.assign({});
                    })
                    .catch(error => {
                        if (typeof error.response.data === 'object') {
                            this.clientErrors = Object.assign({}, error.response.data.errors);
                        }
                    });
            },

            destroyClient(client) {
                Nova.request().delete('/oauth/clients/' + client.id)
                    .then(response => {
                        this.getClients();
                    });
            },

            storeAccessToken: function () {
                this.accessToken = "";
                this.tokenErrors = Object.assign({});
                Nova.request().post(
                        '/oauth/personal-access-tokens',
                        {
                            name: this.tokenName,
                            scopes: this.selectedScopes.map((scope) => {
                                return scope.id;
                            }),
                        }
                    )
                    .then(response => {
                        this.tokenName = '';
                        this.selectedScopes = Object.assign([]);
                        this.tokenErrors = Object.assign({});
                        this.accessToken = response.data.accessToken;
                        this.accessTokens.push(response.data.token);
                    })
                    .catch(error => {
                        this.tokenErrors = ['Something went wrong. Please try again.'];

                        if (typeof error.response.data === 'object') {
                            this.tokenErrors = Object.assign({}, error.response.data.errors);
                        }
                    });
            },

            revokeToken: function (token) {
                Nova.request().delete('/oauth/personal-access-tokens/' + token.id)
                    .then(response => {
                        this.getAccessTokens();
                    });
            },
        },
    }
</script>

<template>
    <div>
        <h1>Passport Manager</h1>
        <h2 class="mt-8 text-90 font-normal text-2xl">Authorized Applications</h2>
        <div
            class="card mt-3"
        >
            <p
                v-show="authorizedTokens.length === 0"
                class="p-6 text-center text-gray-500"
            >
                No OAuth Clients have connected yet.
            </p>
            <div
                v-show="authorizedTokens.length > 0"
                class="relative"
            >
                <div class="w-full overflow-hidden overflow-x-auto relative rounded-lg">
                    <table
                        cellpadding="0"
                        cellspacing="0"
                        data-testid="resource-table"
                        class="w-full border-separate border-spacing-2 border border-slate-400"
                    >
                        <thead>
                            <tr class="bg-gray-50 dark:bg-gray-800">
                                <th class="border border-slate-300 w-16 text-center">ID</th>
                                <th class="border border-slate-300 text-center">Name</th>
                                <th class="border border-slate-300 text-center">Scopes</th>
                                <th class="border border-slate-300 ">&nbsp;</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr
                                v-for="token in authorizedTokens"
                                :key="token.id + '-' + token.client.name"
                                class="bg-white dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
                            >
                                <td class="border border-slate-300">
                                    <span
                                        class="whitespace-no-wrap text-left"
                                        via-resource=""
                                        via-resource-id=""
                                    >
                                        {{ token.id }}
                                    </span>
                                </td>
                                <td class="border border-slate-300">
                                    <span
                                        class="whitespace-no-wrap text-left"
                                        via-resource=""
                                        via-resource-id=""
                                    >
                                        {{ token.client.name }}
                                    </span>
                                </td>
                                <td class="border border-slate-300">
                                    <span
                                        class="whitespace-no-wrap text-left"
                                        via-resource=""
                                        via-resource-id=""
                                    >
                                        {{ token.scopes.join(', ') }}
                                    </span>
                                </td>
                                <td class="border border-slate-300">
                                    <a
                                        class="action-link text-danger"
                                        @click="revokeAuthorizedToken(token)"
                                    >
                                        <svg
                                            class="fill-current h-3 w-3"
                                            aria-hidden="true"
                                            focusable="false"
                                            role="img"
                                            xmlns="http://www.w3.org/2000/svg"
                                            viewBox="0 0 448 512"
                                        >
                                            <path d="M268 416h24a12 12 0 0 0 12-12V188a12 12 0 0 0-12-12h-24a12 12 0 0 0-12 12v216a12 12 0 0 0 12 12zM432 80h-82.41l-34-56.7A48 48 0 0 0 274.41 0H173.59a48 48 0 0 0-41.16 23.3L98.41 80H16A16 16 0 0 0 0 96v16a16 16 0 0 0 16 16h16v336a48 48 0 0 0 48 48h288a48 48 0 0 0 48-48V128h16a16 16 0 0 0 16-16V96a16 16 0 0 0-16-16zM171.84 50.91A6 6 0 0 1 177 48h94a6 6 0 0 1 5.15 2.91L293.61 80H154.39zM368 464H80V128h288zm-212-48h24a12 12 0 0 0 12-12V188a12 12 0 0 0-12-12h-24a12 12 0 0 0-12 12v216a12 12 0 0 0 12 12z"></path>
                                        </svg>
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>


        <h2 class="mt-8 text-90 font-normal text-2xl">OAuth Clients</h2>
        <div class="card mt-3">
            <div class="flex border-b border-40">
                <div class="w-1/5 py-6 px-8">
                    <label
                        for="create-client-name"
                        class="inline-block text-80 pt-2 leading-tight"
                    >
                        Name
                    </label>
                </div>
                <div class="py-6 px-8 w-1/2">
                    <input
                        v-model="clientId"
                        type="hidden"
                    >
                    <input
                        :class="{'border-danger': clientFieldHasError('name')}"
                        v-model="clientName"
                        type="text"
                        placeholder="Name"
                        class="w-full form-control form-input form-input-bordered"
                    >
                    <div
                        v-show="clientFieldHasError('name')"
                        class="help-text error-text mt-2 text-danger"
                        v-text="clientFieldError('name')"
                    ></div>
                    <div class="help-text help-text mt-2">
                        Something your users will recognize and trust.
                    </div>
                </div>
            </div>
            <div class="flex border-b border-40">
                <div class="w-1/5 py-6 px-8">
                    <label
                        for="create-redirect-url"
                        class="inline-block text-80 pt-2 leading-tight"
                    >
                        Redirect URL
                    </label>
                </div>
                <div class="py-6 px-8 w-1/2">
                    <input
                        :class="{'border-danger': clientFieldHasError('redirect')}"
                        v-model="redirectUrl"
                        type="text"
                        placeholder="Redirect URL"
                        class="w-full form-control form-input form-input-bordered"
                    >
                    <div
                        v-show="clientFieldHasError('redirect')"
                        class="help-text error-text mt-2 text-danger"
                        v-text="clientFieldError('redirect')"
                    ></div>
                    <div class="help-text help-text mt-2">
                        Your application's authorization callback URL.
                    </div>
                </div>
            </div>
            <div class="flex items-center">
                <div class="w-1/5 py-3 px-8"></div>
                <div class="pt-6 pb-6 px-8 w-1/2">
                    <button
                        v-show="showUpdateClientButton"
                        type="submit"
                        class="flex-shrink-0 shadow rounded focus:outline-none ring-primary-200 dark:ring-gray-600 focus:ring bg-primary-500 hover:bg-primary-400 active:bg-primary-600 text-white dark:text-gray-800 inline-flex items-center font-bold px-4 h-9 text-sm flex-shrink-0"
                        @click="updateClient"
                    >
                        Update Client
                    </button>
                    <button
                        v-show="showCreateClientButton"
                        type="submit"
                        class="flex-shrink-0 shadow rounded focus:outline-none ring-primary-200 dark:ring-gray-600 focus:ring bg-primary-500 hover:bg-primary-400 active:bg-primary-600 text-white dark:text-gray-800 inline-flex items-center font-bold px-4 h-9 text-sm flex-shrink-0"
                        @click="storeClient"
                    >
                        Create OAuth Client
                    </button>
                </div>
            </div>
        </div>

        <div class="card mt-3 overflow-hidden">
            <p
                v-show="clients.length === 0"
                class="p-6 text-center text-gray-500"
            >
                You have not yet created any OAuth clients.
            </p>

            <table
                v-show="clients.length > 0"
                class="w-full overflow-hidden overflow-x-auto relative rounded-lg"
            >
                <thead>
                    <tr class="bg-gray-50 dark:bg-gray-800">
                        <th class="w-16">ID</th>
                        <th>Name</th>
                        <th>Secret</th>
                        <th class="w-24"></th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="client in clients"
                        :key="'client-' + client.id"
                        class="bg-white dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
                    >
                        <td class="border border-slate-300 text-center">
                            {{ client.id }}
                        </td>
                        <td class="border border-slate-300">
                            {{ client.name }}
                        </td>
                        <td class="border border-slate-300">
                            <code>{{ client.secret }}</code>
                        </td>
                        <td class="border border-slate-300 text-right">
                            <div class="o1-flex o1-items-center o1-justify-end o1-space-x-0 text-gray-400">
                                <a
                                    aria-label="Edit"
                                    class="cursor-pointer toolbar-button hover:o1-text-primary-500 o1-px-2 disabled:o1-opacity-50 disabled:o1-pointer-events-none v-popper--has-tooltip"
                                    tabindex="-1"
                                    @click="editClient(client)"
                                >
                                    <svg
                                        xmlns="http://www.w3.org/2000/svg"
                                        fill="none" viewBox="0 0 24 24"
                                        stroke="currentColor"
                                        width="24" height="24"
                                        class="inline-block"
                                        role="presentation"
                                    >
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path>
                                    </svg>
                                </a>
                                <button
                                    aria-label="Delete"
                                    class="toolbar-button hover:o1-text-primary-500 o1-px-2 disabled:o1-opacity-50 disabled:o1-pointer-events-none v-popper--has-tooltip"
                                    @click="destroyClient(client)"
                                >
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                                         stroke="currentColor" width="24" height="24" class="inline-block"
                                         role="presentation">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                                    </svg>
                                </button>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <h2 class="mt-8 text-90 font-normal text-2xl">Access Tokens</h2>
        <div
            v-show="accessToken.length == 0"
            class="card mt-3"
        >
            <div class="flex border-b border-40">
                <div class="w-1/5 py-6 px-8">
                    <label
                        for="access-token-name"
                        class="inline-block text-80 pt-2 leading-tight"
                    >
                        Name
                    </label>
                </div>
                <div class="py-6 px-8 w-1/2">
                    <input
                        id="access-token-name"
                        :class="{'border-danger': tokenFieldHasError('name')}"
                        v-model="tokenName"
                        type="text"
                        placeholder="Name"
                        class="w-full form-control form-input form-input-bordered"
                    >
                    <div
                        v-show="tokenFieldHasError('name')"
                        class="help-text error-text mt-2 text-danger"
                        v-text="tokenFieldError('name')"
                    ></div>
                </div>
            </div>
            <div
                class="flex border-b border-40"
            >
                <div class="w-1/5 py-6 px-8">
                    <label
                        for="token-scopes"
                        class="inline-block text-80 pt-2 leading-tight"
                    >
                        Scopes
                    </label>
                </div>
                <div class="py-6 px-8 w-1/2">
                    <multiselect
                        id="token-scopes"
                        v-model="selectedScopes"
                        :options="scopes"
                        :multiple="true"
                        :hide-selected="true"
                        :close-on-select="false"
                        :clear-on-select="true"
                        :preserve-search="false"
                        :internal-search="true"
                        :preselect-first="false"
                        :searchable="true"
                        :class="{'border-danger': tokenFieldHasError('scopes')}"
                        placeholder="Select scopes..."
                        label="description"
                        track-by="id"
                    ></multiselect>
                    <div
                        v-show="tokenFieldHasError('scopes')"
                        class="help-text error-text mt-2 text-danger"
                        v-text="tokenFieldError('scopes')"
                    ></div>
                </div>
            </div>
            <div class="flex items-center">
                <div class="w-1/5 py-3 px-8"></div>
                <div class="pt-6 pb-6 px-8 w-1/2">
                    <button
                        type="submit"
                        class="flex-shrink-0 shadow rounded focus:outline-none ring-primary-200 dark:ring-gray-600 focus:ring bg-primary-500 hover:bg-primary-400 active:bg-primary-600 text-white dark:text-gray-800 inline-flex items-center font-bold px-4 h-9 text-sm flex-shrink-0"
                        @click="storeAccessToken"
                    >
                        Create Access Token
                    </button>
                </div>
            </div>
        </div>
        <div
            v-show="accessToken.length > 0"
            class="card mt-3"
        >
            <div class="flex border-b border-40 remove-bottom-border">
                <div class="w-1/5 py-6 px-8">
                    <label
                        for="accessToken"
                        class="inline-block text-80 pt-2 leading-tight"
                    >
                        New Access Token
                    </label>
                </div>
                <div class="py-6 px-8 w-4/5">
                    <textarea
                        id="accessToken"
                        rows="5"
                        class="w-full form-control form-input form-input-bordered py-3 h-auto"
                        v-text="accessToken"
                    ></textarea>
                    <div class="help-text help-text mt-2">
                        Here is your new access token. <strong>This is the only time it will be shown so don't lose it!</strong> You may now use this token to make API requests.
                    </div>
                </div>
            </div>
            <div class="flex items-center">
                <div class="w-1/5 pt-0 pb-6 px-8"></div>
                <div class="pt-0 pb-6 px-8 w-1/2">
                    <button
                        type="submit"
                        class="flex-shrink-0 shadow rounded focus:outline-none ring-primary-200 dark:ring-gray-600 focus:ring bg-primary-500 hover:bg-primary-400 active:bg-primary-600 text-white dark:text-gray-800 inline-flex items-center font-bold px-4 h-9 text-sm flex-shrink-0"
                        @click="clearAccessToken"
                    >
                        Close
                    </button>
                </div>
            </div>
        </div>

        <div
            class="card mt-3"
        >
            <p
                v-show="accessTokens.length === 0"
                class="p-6 text-center text-gray-500"
            >
                You have not yet created any Access Tokens.
            </p>
            <div
                v-show="accessTokens.length > 0"
                class="relative"
            >
                <div class="w-full overflow-hidden overflow-x-auto relative rounded-lg">
                    <table
                        cellpadding="0"
                        cellspacing="0"
                        data-testid="resource-table"
                        class="w-full border-separate border-spacing-2 border border-slate-400"
                    >
                        <thead >
                            <tr class="bg-gray-50 dark:bg-gray-800">
                                <th class="border border-slate-300 text-center">Name</th>
                                <th class="border border-slate-300 text-center">Expires At</th>
                                <th class="border border-slate-300 text-center">Scopes</th>
                                <th class="border border-slate-300 w-16">&nbsp;</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr
                                v-for="token in accessTokens"
                                :key="token.id"
                                class="bg-white dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
                            >
                                <td class="border border-slate-300">
                                    <div class="text-left"
                                         via-resource=""
                                         via-resource-id=""
                                    >
                                        <span
                                            class="whitespace-no-wrap"
                                        >
                                            {{ token.name }}
                                        </span>
                                    </div>
                                </td>
                                <td class="border border-slate-300">
                                    <div class="text-left"
                                         via-resource=""
                                         via-resource-id=""
                                    >
                                        <span
                                            class="whitespace-no-wrap"
                                            via-resource=""
                                            via-resource-id=""
                                        >
                                            {{ token.expires_at }}
                                        </span>
                                    </div>
                                </td>
                                <td class="w-1/2  border border-slate-300">
                                    <div class="w-full overflow-x-auto">
                                        <span
                                            class="whitespace-no-wrap text-right"
                                            via-resource=""
                                            via-resource-id=""
                                        >
                                            {{ token.scopes.join(", ") }}
                                        </span>
                                    </div>
                                </td>
                                <td class="whitespace-no-wrap text-right border border-slate-300">
                                    <a
                                        class="cursor-pointer text-danger"
                                        @click="revokeToken(token)"
                                    >
                                        <svg
                                            class="fill-current"
                                            aria-hidden="true"
                                            focusable="false"
                                            role="img"
                                            xmlns="http://www.w3.org/2000/svg"
                                            viewBox="0 0 448 512"
                                            width="20"
                                            height="20"
                                        >
                                            <path d="M268 416h24a12 12 0 0 0 12-12V188a12 12 0 0 0-12-12h-24a12 12 0 0 0-12 12v216a12 12 0 0 0 12 12zM432 80h-82.41l-34-56.7A48 48 0 0 0 274.41 0H173.59a48 48 0 0 0-41.16 23.3L98.41 80H16A16 16 0 0 0 0 96v16a16 16 0 0 0 16 16h16v336a48 48 0 0 0 48 48h288a48 48 0 0 0 48-48V128h16a16 16 0 0 0 16-16V96a16 16 0 0 0-16-16zM171.84 50.91A6 6 0 0 1 177 48h94a6 6 0 0 1 5.15 2.91L293.61 80H154.39zM368 464H80V128h288zm-212-48h24a12 12 0 0 0 12-12V188a12 12 0 0 0-12-12h-24a12 12 0 0 0-12 12v216a12 12 0 0 0 12 12z"></path>
                                        </svg>
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>
