<template>
  <form action>
    <div class="modal-card" style="width: auto">
      <header class="modal-card-head">
        <p class="modal-card-title">Christmas and New Year Bin Collection Dates 2020/21</p>
        <button aria-label="close" type="button" class="delete" @click="$emit('close')" />
      </header>
      <section class="modal-card-body">
        <table style="width:100%">
          <tbody>
            <tr>
              <th>Your normal collection day</th>
              <th>Your revised Christmas and New Year collection dates</th>
            </tr>
            <tr>
              <td>Monday</td>
              <td>
                <div>Wednesday 30th December</div>
                <div>Wednesday 6th January</div>
                <div>Tuesday 12th January</div>
              </td>
            </tr>
            <tr>
              <td>Tuesday</td>
              <td>
                <div>Thursday 31st December</div>
                <div>Thursday 7th January</div>
                <div>Wednesday 13th January</div>
              </td>
            </tr>
            <tr>
              <td>Wednesday</td>
              <td>
                <div>Saturday 2nd January</div>
                <div>Friday 8th January</div>
                <div>Thursday 14th January</div>
              </td>
            </tr>
            <tr>
              <td>Thursday</td>
              <td>
                <div>Monday 4th January</div>
                <div>Saturday 9th January</div>
                <div>Friday 15th January</div>
              </td>
            </tr>
            <tr>
              <td>Friday</td>
              <td>
                <div>Tuesday 29th December</div>
                <div>Tuesday 5th January</div>
                <div>Monday 11th January</div>
                <div>Saturday 16th January</div>
              </td>
            </tr>
          </tbody>
        </table>
        <div>
            <div class="title is-5" style="padding-top:1rem">Garden Waste Collections</div>
            <div>Garden waste collections due on Friday 25th December will take place on Tuesday 29th December. The Garden Waste Service will then be suspended until week commencing 11th January when collections will take place one day later than your normal collection day</div>
        </div>
      </section>
      <footer class="modal-card-foot">
        <b-button class="button is-primary" @click="$emit('close')">Ok</b-button>
      </footer>
    </div>
  </form>
</template>

<script>
export default {};
</script>

<style scoped>
table,
th,
td {
  border: 1px solid black;
  border-collapse: collapse;
}
th,
td {
  padding: 5px;
  text-align: left;
}
</style>