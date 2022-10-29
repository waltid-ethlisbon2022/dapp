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

                                                                                 
                        <p class="lead connect-txt mt-6">
                            Start Wallet Connect Chat/SSI protocol and scan the QR code in order to connect your wallet. </p>


                        <button class="wallect-connect-btn btn btn-primary  my-2 fw-bold _btn" @click="initWalletConnectChat">Start Protocol</button>
<!-- 
                        <p >
                            <a :href="'/xdevice/'"
                                class="wallect-connect-btn btn btn-primary my-2 fw-bold _btn">Connect Wallet using SSI over WalletConnect</a>
                        </p> -->
                    
                        <div v-show="protocolLog.length > 0" class="text-center">
                            <canvas :id="'qr-code'" />
                        </div>


                        <div class="protocolLog mt-3">

                            <ul>
                                <li v-for="line in protocolLog">{{ line }}</li>
                            </ul>
                                
                        </div>           
                     

                        <p class="text-muted fw-bold">© 2022 walt.id</p>
                    </div>
                </div>
            </section>
        </main>
    </div>
</template>

<script>
import { INVALID_REQUEST } from '@walletconnect/jsonrpc-utils';
import {config} from '/config.js'
import QRious from "qrious"

if (typeof window !== "undefined")
     window.global = window;
const ChatClient = require("@walletconnect/chat-client").ChatClient

export default {
    data() {
        return {
            protocolLog: []
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

            this.protocolLog.push(`Initializing WalletConnect Chat Client for ${dappAccount}`)
            console.log(this.protocolLog)

            console.log(config.walletConnectId)
        
            const chatClient = await ChatClient.init({
                projectId: config.walletConnectId
            })

            const retReg = await chatClient.register({ account: dappAccount });

            this.protocolLog.push(`Register client returned: ${retReg}`)
            console.log(this.protocolLog);

            chatClient.on("chat_invite", async (event) => {
                this.protocolLog.push("Client received invite");
                
                console.log(event)
        
                const topic = await chatClient.accept({id: event.id}).catch(e => {console.error(e)})

                this.protocolLog.push(`Client accepted. topic: ${topic}`);

                await chatClient.message({
                    topic: topic, // Topic of the chat thread this message should be sent to.
                    payload: {
                        message: "VerifiableId", // the message you want to send.
                        authorAccount: dappAccount, // your CAIP-2 formatted account that you registered previously.
                        timestamp: Date.now(),
                    },
                }).catch(e => {console.error(e)});
                
            });

            chatClient.on("chat_joined", async (event) => {
                console.log("Client joined");
            });

            chatClient.on("chat_ping", async (event) => {
                console.log("chat_ping joined");
            });


            chatClient.on("chat_message", async (event) => {
                console.log(event);
                console.log( "Credential received: " + event.params.message);
                this.protocolLog.push("Credential received: " + event.params.message);
               // const sessionInfo = await this.$axios.$get("/api/wallet/presentation/create?type=" + event.params.message)
                //this.$router.replace("/CredentialRequest/?sessionId=" + sessionInfo.id)
                
            });

        },
    },
    mounted() {
            new QRious({
                element: document.getElementById('qr-code'),
                value: `eip155:1:${config.ethAccount}`,
                size: 300
            })
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

.protocolLog {
    background-color: black;
    color:greenyellow;
    text-align: left;
}
</style>
