<template>
    <div>
        <header>
            <Notice
                text=": Wallet Connect meets Self-Sovereign Identity">
            </Notice>
        </header>
        <main class="_home d-flex justify-content-centr align-items-center">
            <section class="py-5 text-center container">
                <div class="row py-lg-5">
                    <div class="col-lg-6 col-md-8 mx-auto">
                        <img class="p-5" src="https://walletconnect.com/images/logo.svg" alt="Logo" />
                        <p >
                            <a :href="'/xdevice/' + encodeURIComponent(vidSchemaUri)"
                                class="wallect-connect-btn btn btn-primary  my-2 fw-bold _btn">Connect Wallet using SSI over WalletConnect</a>
                        </p>
                                                         
                        <p class="lead connect-txt mt-6">
                            Connect your wallet and share<br>your credentials to access services. </p>
                     
                        <button class="btn btn-primary" @click="initWalletConnectChat">Start Client</button>
                        <p class="text-muted fw-bold">© 2022 walt.id</p>
                    </div>
                </div>
            </section>
        </main>
    </div>
</template>

<script>
import {config} from '/config.js'

if (typeof window !== "undefined")
     window.global = window;
const ChatClient = require("@walletconnect/chat-client").ChatClient

export default {
    data() {
        return {
            vidSchemaUri1: "https://ngiatlantic.info/schema/V-2022-1/OpenBadgeCredential.json",
            vidSchemaUri: 'https://api.preprod.ebsi.eu/trusted-schemas-registry/v1/schemas/0xb77f8516a965631b4f197ad54c65a9e2f9936ebfb76bae4906d33744dbcc60ba',
            bidSchemaUri: 'https://raw.githubusercontent.com/walt-id/waltid-ssikit-vclib/master/src/test/resources/schemas/EuropeanBankIdentity.json'
        }
    },
    async asyncData({ $axios }) {
        const wallets = await $axios.$get('/verifier-api/wallets/list')
        return { wallets }
    },
    methods: {
        encodeURIComponent(str) {
            return encodeURIComponent(str)
        },
        initWalletConnectChat: async function() {

            let dappAccount = `eip155:1:${config.ethAccount}`
            console.log(`Initializing WalletConnect Chat Client for ${dappAccount} ...`)

            console.log(config.walletConnectId)
        
            const chatClient = await ChatClient.init({
                projectId: config.walletConnectId
            })

            await chatClient.register({ account: `${dappAccount}` });

        }
    }
}
</script>

<style scoped>
._home {
    height: 100vh;
    background: url("https://i.ibb.co/0VTB7mP/bcg-background.jpg") no-repeat center fixed;
    background-size: cover;
}

._btn {
    font-size: 18px;
    padding: 10px 55px;
}

.connect-txt {
    color: white;
}

.wallect-connect-btn{

    background-color: transparent;
     border-color: white!important;
}
</style>
