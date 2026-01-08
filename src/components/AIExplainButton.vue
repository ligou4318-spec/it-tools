<script setup lang="ts">
import { computed, ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { IconSparkles, IconClock, IconCode, IconBriefcase } from '@tabler/icons-vue';

interface Props {
  cronExpression: string;
  isValid: boolean;
}

const props = defineProps<Props>();
const { t } = useI18n();

const showExplanation = ref(false);
const isLoading = ref(false);

const aiExplanation = computed(() => {
  if (!props.isValid || !props.cronExpression) {
    return null;
  }

  const expr = props.cronExpression.trim();
  const parts = expr.split(/\s+/);

  // AI-Powered smart explanation with US business context
  return generateAIExplanation(expr, parts);
});

function generateAIExplanation(expr: string, parts: string[]) {
  const explanations: {
    plainEnglish: string;
    businessScenario: string;
    codeExample: string;
    timezoneTip: string;
  } = {
    plainEnglish: '',
    businessScenario: '',
    codeExample: '',
    timezoneTip: '',
  };

  // Smart explanation logic
  if (expr.includes('@yearly') || expr.includes('@annually')) {
    explanations.plainEnglish = "Runs once a year, on January 1st at midnight.";
    explanations.businessScenario = "💼 Perfect for annual tasks like:\n• Sending year-end reports\n• Renewing subscriptions\n• Annual database cleanup\n• New Year greetings";
    explanations.codeExample = `// Example: Annual subscription reminder
cron.schedule('@yearly', () => {
  User.sendAnnualRenewalNotice();
  Billing.generateYearlyReport();
});`;
    explanations.timezoneTip = "🇺🇸 Runs at midnight in your server timezone. For US users:\n• EST (New York): 12:00 AM\n• PST (California): 9:00 PM (previous day)";
  }
  else if (expr.includes('@monthly')) {
    explanations.plainEnglish = "Runs on the first day of every month at midnight.";
    explanations.businessScenario = "💼 Ideal for monthly cycles:\n• Generating invoices\n• Monthly billing statements\n• Sending payroll\n• Monthly KPI reports";
    explanations.codeExample = `// Example: Monthly invoice generation
cron.schedule('@monthly', () => {
  Billing.generateInvoices();
  Accounting.closeMonthlyBooks();
});`;
    explanations.timezoneTip = "🇺🇸 First of the month at midnight:\n• EST: 12:00 AM ET\n• PST: 9:00 PM PT (previous day)";
  }
  else if (expr.includes('@weekly')) {
    explanations.plainEnglish = "Runs once a week on Sunday at midnight.";
    explanations.businessScenario = "💼 Great for weekly tasks:\n• Weekly summary emails\n• Database maintenance\n• Sprint retrospectives\n• Weekly backup jobs";
    explanations.codeExample = `// Example: Weekly reports
cron.schedule('@weekly', () => {
  Analytics.generateWeeklyReport();
  Users.sendWeeklyDigest();
});`;
    explanations.timezoneTip = "🇺🇸 Sunday midnight:\n• EST: 12:00 AM Sunday\n• PST: 9:00 PM Saturday";
  }
  else if (expr.includes('@daily') || expr.includes('@midnight')) {
    explanations.plainEnglish = "Runs every day at midnight.";
    explanations.businessScenario = "💼 Perfect for daily operations:\n• Daily data backups\n• Resetting counters/metrics\n• Daily report generation\n• Cache cleanup";
    explanations.codeExample = `// Example: Daily maintenance
cron.schedule('@daily', () => {
  Database.backup();
  Cache.clearExpired();
  Logs.rotate();
});`;
    explanations.timezoneTip = "🇺🇸 Daily at midnight:\n• EST: 12:00 AM\n• PST: 9:00 PM (previous day)";
  }
  else if (expr.includes('@hourly')) {
    explanations.plainEnglish = "Runs every hour at the top of the hour.";
    explanations.businessScenario = "💼 Good for hourly processes:\n• Stock price updates\n• Social media polling\n• Cache warming\n• Hourly data sync";
    explanations.codeExample = `// Example: Hourly sync
cron.schedule('@hourly', () => {
  APIs.syncExternalData();
  Queue.processPending();
});`;
    explanations.timezoneTip = "🇺🇸 Every hour at :00 minutes";
  }
  else {
    // Custom expressions - smart parsing
    const [minute, hour, day, month, weekday] = parts.slice(-5);

    explanations.plainEnglish = generateCustomExplanation(minute, hour, day, month, weekday);
    explanations.businessScenario = generateBusinessScenario(minute, hour, day, month, weekday);
    explanations.codeExample = generateCodeExample(minute, hour);
    explanations.timezoneTip = "🇺🇸 Times shown in your server timezone. Consider US timezones if serving American users.";
  }

  return explanations;
}

function generateCustomExplanation(min: string, hr: string, day: string, month: string, wd: string): string {
  let explanation = "This cron job ";

  // Minute
  if (min === '*') {
    explanation += "runs every minute";
  } else if (min.includes('*/')) {
    const interval = min.split('/')[1];
    explanation += `runs every ${interval} minutes`;
  } else if (min.includes(',')) {
    const mins = min.split(',').join(', ');
    explanation += `runs at minutes ${mins}`;
  } else if (min.includes('-')) {
    const [start, end] = min.split('-');
    explanation += `runs from minute ${start} through ${end}`;
  } else {
    explanation += `runs at minute ${min}`;
  }

  // Hour
  if (hr !== '*') {
    if (hr.includes('*/')) {
      const interval = hr.split('/')[1];
      explanation += `, every ${interval} hours`;
    } else {
      explanation += ` of hour ${hr}`;
    }
  }

  explanation += ". Check the detailed breakdown below for more specifics.";
  return explanation;
}

function generateBusinessScenario(min: string, hr: string, day: string, month: string, wd: string): string {
  const scenarios: string[] = [];

  // Business hours detection (9 AM - 5 PM EST)
  if ((parseInt(hr) >= 9 && parseInt(hr) <= 17) || hr.includes('*/')) {
    scenarios.push("• Business hours operation (good for user-facing tasks)");
  }

  // Night jobs
  if (parseInt(hr) >= 0 && parseInt(hr) <= 5) {
    scenarios.push("• After-hours processing (ideal for batch jobs, backups)");
  }

  // Start of day
  if (hr === '8' || hr === '9') {
    scenarios.push("• Morning routine (daily reports, morning emails)");
  }

  // End of day
  if (hr === '17' || hr === '18') {
    scenarios.push("• End-of-day tasks (daily summaries, close-of-business)");
  }

  // Weekend vs weekday
  if (wd === '0' || wd === '6' || wd === '7') {
    scenarios.push("• Weekend maintenance (system updates, heavy processing)");
  } else if (wd === '1-5') {
    scenarios.push("• Weekday operation (business days only)");
  }

  if (scenarios.length === 0) {
    scenarios.push("• Custom schedule - optimize based on your specific needs");
  }

  return "💼 Business Use Cases:\n" + scenarios.join('\n');
}

function generateCodeExample(min: string, hr: string): string {
  const hourStr = hr === '*' ? '*' : hr;
  const minStr = min === '*' ? '*' : min;

  return `// Cron job: ${minStr} ${hourStr} * * *
// Copy this code to implement:

const cron = require('node-cron');

cron.schedule('${min} ${hr} * * *', () => {
  // Your task here
  console.log('Running scheduled task...');

  // Example tasks:
  // - Data synchronization
  // - Email notifications
  // - Database cleanup
  // - Report generation
});

// Pro tip: Use process managers like PM2
// to keep your cron jobs running in production`;
}

async function showAIExplanation() {
  isLoading.value = true;
  // Simulate AI processing time for better UX
  await new Promise(resolve => setTimeout(resolve, 600));
  showExplanation.value = true;
  isLoading.value = false;
}

function closeExplanation() {
  showExplanation.value = false;
}
</script>

<template>
  <div class="ai-explain-container">
    <n-button
      v-if="!showExplanation"
      type="primary"
      :loading="isLoading"
      :disabled="!isValid"
      @click="showAIExplanation"
      size="small"
      quaternary
    >
      <template #icon>
        <n-icon :component="IconSparkles" />
      </template>
      {{ t('aiExplain.button') }}
    </n-button>

    <n-collapse-transition>
      <div v-if="showExplanation && aiExplanation" class="ai-explanation-box">
        <n-card
          title="✨ AI-Powered Explanation"
          closable
          @close="closeExplanation"
          size="small"
        >
          <!-- Plain English -->
          <div class="explanation-section">
            <h4><n-icon :component="IconBriefcase" /> {{ t('aiExplain.plainEnglish') }}</h4>
            <p>{{ aiExplanation.plainEnglish }}</p>
          </div>

          <n-divider />

          <!-- Business Scenarios -->
          <div class="explanation-section">
            <h4><n-icon :component="IconBriefcase" /> {{ t('aiExplain.businessScenarios') }}</h4>
            <pre class="scenario-text">{{ aiExplanation.businessScenario }}</pre>
          </div>

          <n-divider />

          <!-- Code Example -->
          <div class="explanation-section">
            <h4><n-icon :component="IconCode" /> {{ t('aiExplain.codeExample') }}</h4>
            <pre class="code-example">{{ aiExplanation.codeExample }}</pre>
          </div>

          <n-divider />

          <!-- Timezone Tip -->
          <div class="explanation-section">
            <h4><n-icon :component="IconClock" /> {{ t('aiExplain.timezoneTip') }}</h4>
            <p class="timezone-text">{{ aiExplanation.timezoneTip }}</p>
          </div>
        </n-card>
      </div>
    </n-collapse-transition>
  </div>
</template>

<style lang="less" scoped>
.ai-explain-container {
  margin-top: 10px;
}

.ai-explanation-box {
  margin-top: 10px;
  animation: slideDown 0.3s ease-out;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.explanation-section {
  h4 {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-bottom: 10px;
    color: #18a058;
    font-weight: 600;
  }

  p {
    line-height: 1.6;
    color: #333;
  }
}

.scenario-text {
  background: #f5f5f5;
  padding: 12px;
  border-radius: 6px;
  border-left: 3px solid #18a058;
  white-space: pre-wrap;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  font-size: 14px;
  line-height: 1.6;
  color: #2c3e50;
}

.code-example {
  background: #1e1e1e;
  color: #d4d4d4;
  padding: 16px;
  border-radius: 6px;
  overflow-x: auto;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.5;
}

.timezone-text {
  background: #e3f2fd;
  padding: 10px;
  border-radius: 6px;
  border-left: 3px solid #2196f3;
  font-size: 14px;
}
</style>
