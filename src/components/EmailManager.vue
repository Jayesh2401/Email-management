<template>
  <div class="container">
    <h1>Email Management</h1>

    <div class="lists-container">
      <div class="list-section">
        <h3>Available Recipients</h3>

        <div class="input-section">
          <a-auto-complete
            v-model:value="inputValue"
            placeholder="Search domain or enter domain"
            class="autocomplete"
          />
          <a-button @click="addEmail" type="primary" :disabled="!isValidEmail">
            Add
          </a-button>
        </div>
        <template v-for="(emails, domain) in filteredDomains" :key="domain">
          <a-collapse v-if="emails.length > 1" accordion>
            <a-collapse-panel :header="domain">
              <a-checkbox
                :checked="isAllSelected(domain, 'available')"
                @change="() => toggleDomainSelection(domain, 'available')"
              >
                Select All
              </a-checkbox>
              <a-list bordered>
                <a-list-item v-for="email in emails" :key="email.email">
                  <a-checkbox
                    :checked="email.isSelected"
                    @change="() => moveToSelected(email)"
                  >
                    {{ email.email }}
                  </a-checkbox>
                </a-list-item>
              </a-list>
            </a-collapse-panel>
          </a-collapse>

          <div v-else>
            <a-list bordered>
              <a-list-item v-for="email in emails" :key="email.email">
                <a-checkbox
                  :checked="email.isSelected"
                  @change="() => moveToSelected(email)"
                >
                  {{ email.email }}
                </a-checkbox>
              </a-list-item>
            </a-list>
          </div>
        </template>
      </div>

      <div class="list-section">
        <h3>Selected Recipients</h3>

        <template v-for="(emails, domain) in selectedDomain" :key="domain">
          <a-collapse v-if="emails.length > 1" accordion>
            <a-collapse-panel :header="domain">
              <a-checkbox
                :checked="isAllSelected(domain, 'selected')"
                @change="() => toggleDomainSelection(domain, 'selected')"
              >
                Select All
              </a-checkbox>
              <a-list bordered>
                <a-list-item v-for="email in emails" :key="email.email">
                  <a-checkbox
                    :checked="email.isSelected"
                    @change="() => moveToAvailable(email)"
                  >
                    {{ email.email }}
                  </a-checkbox>
                </a-list-item>
              </a-list>
            </a-collapse-panel>
          </a-collapse>

          <div v-else>
            <a-list bordered>
              <a-list-item v-for="email in emails" :key="email.email">
                <a-checkbox
                  :checked="email.isSelected"
                  @change="() => moveToAvailable(email)"
                >
                  {{ email.email }}
                </a-checkbox>
              </a-list-item>
            </a-list>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent, ref, computed } from "vue";
import {
  AutoComplete,
  List,
  Button,
  Checkbox,
  Collapse,
  message,
} from "ant-design-vue";

export default defineComponent({
  components: {
    "a-auto-complete": AutoComplete,
    "a-list": List,
    "a-list-item": List.Item,
    "a-button": Button,
    "a-checkbox": Checkbox,
    "a-collapse": Collapse,
    "a-collapse-panel": Collapse.Panel,
  },
  setup() {
    const emailData = ref([
      { email: "ann@timescale.com", isSelected: false },
      { email: "bob@timescale.com", isSelected: false },
      { email: "brian@qwerty.com", isSelected: false },
      { email: "james@qwerty.com", isSelected: false },
      { email: "jane@awesome.com", isSelected: false },
      { email: "kate@qwerty.com", isSelected: true },
      { email: "mike@hello.com", isSelected: false },
      { email: "try@alone.com", isSelected: false },
    ]);

    const inputValue = ref("");

  

    const domains = computed(() => {
      const domainMap = {};
      emailData.value.forEach(({ email, isSelected }) => {
        const domain = email.split("@")[1];
        if (!domainMap[domain]) domainMap[domain] = [];
        domainMap[domain].push({ email, isSelected });
      });
      return domainMap;
    });

    const filteredDomains = computed(() => {
    if (!inputValue.value) return domains.value; 

    const filtered = {};
    Object.entries(domains.value).forEach(([domain, emails]) => {
      const matchedEmails = emails.filter((emailObj) =>
        emailObj.email.toLowerCase().includes(inputValue.value.toLowerCase())
      );
      if (matchedEmails.length > 0) {
        filtered[domain] = matchedEmails;
      }
    });
    return filtered;
  });

    // Separate selected emails grouped by domain
    const selectedDomain = computed(() => {
      const domainMap = {};
      emailData.value.forEach(({ email, isSelected }) => {
        if (isSelected) {
          const domain = email.split("@")[1];
          if (!domainMap[domain]) domainMap[domain] = [];
          domainMap[domain].push({ email, isSelected });
        }
      });
      return domainMap;
    });

    const moveToSelected = (email) => {
      if (!email.isSelected) {
        email.isSelected = true;
        const index = emailData.value.findIndex((e) => e.email === email.email);
        if (index !== -1)
          emailData.value[index] = { ...email, isSelected: true };
      } else {
        email.isSelected = false;
        const index = emailData.value.findIndex((e) => e.email === email.email);
        if (index !== -1)
          emailData.value[index] = { ...email, isSelected: false };
      }
    };

    const moveToAvailable = (email) => {
      if (email.isSelected) {
        email.isSelected = false;
        const index = emailData.value.findIndex((e) => e.email === email.email);
        if (index !== -1)
          emailData.value[index] = { ...email, isSelected: false };
      }
    };

    const toggleDomainSelection = (domain, type) => {
      const domainEmails =
        type === "selected"
          ? selectedDomain.value[domain]
          : domains.value[domain];
      const allSelected = domainEmails.every((email) => email.isSelected);

      domainEmails.forEach((email) => {
        email.isSelected = !allSelected;
        const index = emailData.value.findIndex((e) => e.email === email.email);
        if (index !== -1)
          emailData.value[index] = { ...email, isSelected: !allSelected };
      });
    };

    const isAllSelected = (domain, type) => {
      const domainEmails =
        type === "selected"
          ? selectedDomain.value[domain]
          : domains.value[domain];
      return domainEmails.every((email) => email.isSelected);
    };

    const addEmail = () => {
      if (!/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(inputValue.value)) {
        message.error("Invalid email format");
        return;
      }
      if (!emailData.value.some((e) => e.email === inputValue.value)) {
        emailData.value.push({ email: inputValue.value, isSelected: false });
      }
      inputValue.value = "";
    };

    const isValidEmail = computed(() => {
      return /^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(inputValue.value);
    });

    return {
      emailData,
      inputValue,
      domains,
      selectedDomain,
      moveToSelected,
      moveToAvailable,
      toggleDomainSelection,
      isAllSelected,
      addEmail,
      filteredDomains,
      isValidEmail
    };
  },
});
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: auto;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #1890ff;
  margin-bottom: 20px;
}

.input-section {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.autocomplete {
  flex-grow: 1;
}

.lists-container {
  display: flex;
  gap: 20px;
}

.list-section {
  flex: 1;
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
}
</style>
