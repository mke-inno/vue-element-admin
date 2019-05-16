<template>
  <create-campaign :is-edit="false" />
</template>

<script>
import CreateCampaign from './components/CreateCampaign'

export default {
  name: 'CreateForm',
  components: { CreateCampaign }
}
</script>

