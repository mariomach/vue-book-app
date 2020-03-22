<template>
<div>
    <b-alert variant="success" show> {{ message }}</b-alert>
    <p>It works!</p>
    <br>
</div>
</template>

<script>
export default {
    props: ['message'],
};
</script>