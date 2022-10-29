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
                            Start Wallet Connect Chat/SSI protocol <br /> and scan the QR code in order to connect your wallet and exchange W3C Verifiable Credentials. </p>


                        <button class="wallect-connect-btn btn btn-primary  my-2 fw-bold _btn" @click="initWalletConnectChat">Start Protocol</button>
<!-- 
                        <p >
                            <a :href="'/xdevice/'"
                                class="wallect-connect-btn btn btn-primary my-2 fw-bold _btn">Connect Wallet using SSI over WalletConnect</a>
                        </p> -->
                    
                        <!-- <div v-show="protocolLog.length > 0" class="text-center">
                            <canvas :id="'qr-code'" />
                        </div> -->

                        <div class="userProfile">
                            <img class="profilePicture" src="https://avatars.githubusercontent.com/u/55081379?v=4" />
                            <br />
                            <span><i class="bi bi-check"></i> First Name: Phil </span><br />
                            <span><i class="bi bi-check"></i> Last Name: Peace </span><br />
                            <span><i class="bi bi-check"></i> Address: Some Street 45</span><br />
                            <span><i class="bi bi-check"></i> Country: Austria </span><br />
                            <span><i class="bi bi-check"></i> Email: phil@example.xyz </span><br />
                            <span><i class="bi bi-check"></i> Tel.nr.: 0043 1234567 </span><br />
                        </div>


                        <div class="protocolLog mt-3  overflow-hidden ...">

                            <ul>
                                <li v-for="line in protocolLog" :key="line">{{ line }}</li>
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
                const verificationResponse = await this.$axios.$post("/verifier-api/verify/walletconnect", event.params.message)
                console.log(verificationResponse)
                this.protocolLog.push("Verification result is valid: " + verificationResponse.isValid)
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
    background-color: darkgreen;
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

.userProfile {
    margin: 10pt;
    color: white;
}

.profilePicture {
    width: 40%;
    border-radius: 50%;
}
</style>
