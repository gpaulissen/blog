---
title: Contact
permalink: /contact/
sitemap: false
---
Please leave your email address and comment here.

<script src="https://www.google.com/recaptcha/api.js" async defer></script>
<form action="{{ site.contact.endpoint }}" method="POST">
    <label>{{ site.data.ui-text[site.locale].comment_form_email_label }}
        <input type="email" id="email" name="email" required>
    </label>
    <label>{{ site.data.ui-text[site.locale].comment_form_comment_label }}
        <textarea rows="4" cols="50" name="comments"></textarea>
    </label>
    <input type="hidden" name="_gotcha">
    <div class="g-recaptcha" data-sitekey="{{ site.contact.recaptcha.sitekey }}"></div>
    <button type="submit" class="btn">{{ site.data.ui-text[site.locale].comment_btn_submit }}</button>
</form>
