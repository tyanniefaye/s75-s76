<template>
	<h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
            <div class="contact-section">
                <div class="row align-items-center mt-4">
                    <div class="col-md-6 map-container">
                        <iframe id="gmap_canvas" src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe>
                    </div>
                    <div class="col-md-6">
                        <form @submit.prevent="submitForm">
                            <div class="mb-3">
                                <input type="text" class="form-control contact-form-control" placeholder="First Name M.I. Last Name" v-model="name" required>
                            </div>
                            <div class="mb-3">
                                <input type="email" class="form-control contact-form-control" placeholder="Email" v-model="email" required>
                            </div>
                            <div class="mb-3">
                                <textarea class="form-control contact-form-control" rows="6" placeholder="Message" v-model="message" required></textarea>
                            </div>
                            <div class="form-footer">
                                <div class="social-icons">
                                    <a href="https://www.linkedin.com/in/charles-babbage-8291a6211/" id="linkedin"><i class="fab fa-linkedin"></i></a>
                                    <a href="https://gitlab.com/cbabbage0991" id="gitlab"><i class="fab fa-gitlab"></i></a>
                                    <a href="https://github.com/cbabbage0991" id="github"><i class="fab fa-github"></i></a>
                                </div>
                                <button type="submit" class="submit-btn pl-5 pr-5" :disabled="isLoading">{{isLoading ? "Sending..." : "Submit"}}</button>
                            </div>
                        </form>
                        
                    </div>
                </div>
            </div>

</template>

<script setup>
    
    import { ref, onMounted, onBeforemount } from 'vue';
    import { Notyf } from 'notyf';
    import 'notyf/notyf.min.css';

    const notyf = new Notyf();

    const WEB3FORMS_ACCESS_KEY = "84c4e636-7fd4-4cc3-87a8-89fd056380df";

    const subject = "New message from Portfolio Contact Form";

    const name = ref("");
    const email = ref("");
    const message = ref("");

    const isLoading = ref(false);


    const submitForm = async() => {

        if(!recaptchaToken.value){
            notyf.error("Please verify that you are not a robot.");
            return;
        }

        isLoading.value = true;

        try {

            const response = await fetch("https://api.web3forms.com/submit", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Accept: "application/json",
                },
                body: JSON.stringify({
                    access_key: WEB3FORMS_ACCESS_KEY,
                    subject: subject,
                    name: name.value,
                    email: email.value,
                    message: message.value
                })
            })
            const result = await response.json();

            if (result.success){
                console.log(result);
                isLoading.value = false;
                notyf.success("Message sent");
            }
        } catch (error){
            console.log(error);
            isLoading.value = false;
            notyf.error("Failed to send message");
        } finally {
            restRecaptcha();
        }
    }


    const SITE_KEY = '6Lds7gksAAAAAFQh0WxWmwWH3Dypkf50IbWr2hvS'

    const recaptchaContainer = ref(null);
    const recaptchaWidgetId = ref(null);
    const recaptchaToken = ref(null);

    function onRecaptchaSuccess(token){
        recaptchaToken.value = token;
    }

    function onRecaptchaExpired(){
        recaptchaToken.value = '';
    }

    function renderRecaptcha(){
        if(!window.grecaptcha){
            console.error('recaptcha not loaded');
            return;
        }
        recaptchaWidgetId.value = window.grecaptcha.render(recaptcha.value, {
            sitekey: SITE_KEY,
            size: 'normal',
            callback: onRecaptchaSuccess,
            'expired-callback': onRecaptchaExpired
        });
    }

    function resetRecaptcha(){
        if(recaptchaWidgetId.value !==null){
            window.grecaptcha.reset(recaptchaWidgetId.value);
            recaptchaToken.value = '';
        }
    }

    onMounted(() => {
        const interval = setInterval(() => {
            if(window.grecaptcha && window.grecaptcha.render){
                renderRecaptcha();
                clearInterval(interval)
            }
        }, 100);

        onBeforemount(() => {
            clearInterval(interval)
        })
    });

</script>