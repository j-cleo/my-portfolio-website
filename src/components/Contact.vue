<script setup>
    import { ref, onMounted, onBeforeUnmount } from 'vue';
    import axios from 'axios';
    import { Notyf } from 'notyf';
    import 'notyf/notyf.min.css';

    const notyf = new Notyf();
    
    const WEB3FORMS_ACCESS_KEY = "92a88244-cd1e-4515-bdcd-894201c44a53";
    const SITE_KEY = "6LcEkBouAAAAALAct_ry_QRON9acbgLx3d-qy4bh";
    const SECRET_KEY = "6LcEkBouAAAAAEp_jYFLczmRqEwLbnYGVD4I1n2a";

    const subject = "New Message From Portfolio Contact Form";

    const name = ref("");
    const email = ref("");
    const message = ref("");

    const isLoading = ref(false);

    const recaptchaContainer = ref(null);
    const recaptchaWidgetId = ref(null);
    const recaptchaToken = ref('');

    // Callback called by reCAPTCHA when successful
    function onRecaptchaSuccess(token) {
      recaptchaToken.value = token;
    }

    // Callback when expired
    function onRecaptchaExpired() {
      recaptchaToken.value = '';
    }

    // Function to render the reCAPTCHA widget
    function renderRecaptcha() {
      if (!window.grecaptcha) {
        console.error('reCAPTCHA not loaded');
        return;
      }

      recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
        sitekey: SITE_KEY,
        size: 'normal', // or 'compact'
        callback: onRecaptchaSuccess,
        'expired-callback': onRecaptchaExpired,
      });
    }

    // Function to reset reCAPTCHA 
    function resetRecaptcha() {
      if (recaptchaWidgetId.value !== null) {
        window.grecaptcha.reset(recaptchaWidgetId.value);
        recaptchaToken.value = '';
      }
    }



    onMounted(() => {
      // This code waits for the Google reCAPTCHA library to load, then renders the reCAPTCHA widget using onMounted hook. 
      // The widget is rendered with grecaptcha.render(), which requires a sitekey. 
      // Callback functions handle success and expiration events. 
      // reCAPTCHA is reset upon form submission to clear the token.
      const interval = setInterval(() => {
        if (window.grecaptcha && window.grecaptcha.render) {
          renderRecaptcha();
          clearInterval(interval);
        }
      }, 100);

      onBeforeUnmount(() => {
        clearInterval(interval);
      });
    });

    const submitForm = async() => {
        if (!recaptchaToken.value) {
            notyf.error('Please verify that you are not a robot.');
            return;
        }
        isLoading.value = true;

        try {
            const response = await axios.post("https://api.web3forms.com/submit", {
                access_key: WEB3FORMS_ACCESS_KEY,
                subject: subject,
                name: name.value,
                email: email.value,
                message: message.value
            }, {
                headers: {
                    'Content-Type' : "application/json",
                    Accept: "application/json"
                }
            }
        )

        const result = response.data;
        if(result.success) {
            notyf.success("Message Sent!");
            name.value = "";
            email.value = "";
            message.value = "";
        }

        } catch (e) {
            notyf.error("Failed to send email!");
        } finally {
            isLoading.value = false;
        }
    }
</script>

<template>
	<section id="contact" class="py-5">
				<div class="row g-0 justify-content-center mx-auto contact-wrapper">
					<div class="section-header d-flex flex-column align-items-center">
						<h3 class="inter-bold rounded-pill">Contact</h3>
						<h6 class="pt-sans-regular">Want to connect and talk? Fill up the form to send me a message!</h6>
					</div>
					<div class="col-12 col-md-6 contact-form px-4 px-lg-5 py-5 d-flex flex-column justify-content-center">
						<form @submit.prevent="submitForm">
							<div class="mb-4">
								<label for="contact-name" class="form-label">Name</label>
								<input v-model="name" type="text" class="form-control py-3" id="contact-name" placeholder="John Doe">
							</div>
							<div class="mb-4">
								<label for="contact-email" class="form-label">Email Address</label>
								<input v-model="email" type="email" class="form-control py-3" id="contact-email" placeholder="john.doe@email.com">
							</div>
							<div class="mb-4">
								<label for="contact-message" class="form-label">Message</label>
								<textarea v-model="message" class="form-control py-3" id="contact-message" rows="12"></textarea>
							</div>
							<div class="text-end pt-4 pb-5">
								<button type="submit" class="btn-submit px-4" :disabled="isLoading">{{ isLoading ? "Sending . . . " : "Submit" }}</button>
							</div>
						</form>
						<div class="d-flex justify-content-end mt-2">
                            <div ref="recaptchaContainer"></div>
                        </div>
					</div>
				</div>
			</section>
</template>