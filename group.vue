<template>
  <navigation/>
  <div class="main-layout">
    <div class="top-row"> 
  <div class="budget-container">
    <div class="budget-content">
      <div v-if="budgetSuccessMessage" class="budget-success-message" :class="{ hide: budgetHideMessage }">{{ budgetSuccessMessage }}</div>
      <div class="budget-header"> <!--NEWWWWWWWWWW-->
      <h3>Monthly Budget</h3>
      <button v-if="!hasExistingBudget" @click="showAddBudgetForm" class="add-budget-btn">Add Budget</button>
      <button @click="showEditBudgetForm" class="edit-budget-btn">Edit Budget</button>
    </div>

      <div v-if="!isAddingBudget && !isEditingBudget" class="budget-display">
            <div class="budget-info">
              <div class="budget-month-row"> <!--NEWWWWWWWWWWW-->
                <span class="budget-label">Month-Year:</span>
              <span class="budget-month">{{ formatMonthYear(selectedMonthYear) }}</span>
              </div>
              <div class="budget-amount-row">
                <span class="budget-label">Budget Amount:</span>
              <span class="budget-amount">{{ formatPHP(currentBudgetAmount) }}</span>
            </div>
        </div>
        </div>

        <!--FOR ADDING BUDGET-->
        <div v-if="isAddingBudget" class="budget-form">
        <div class="form-group">
          <label for="monthYear">Month-Year:</label>
          <select id="monthYear" v-model="selectedMonthYear">
            <option v-for="month in availableMonths" :key="month" :value="month">{{ formatMonthYear(month) }}</option>
          </select>
        </div>
        <div class="form-group">
          <label for="budgetAmount">Budget Amount (₱):</label>
          <input type="number" id="budgetAmount" v-model="budgetAmount" placeholder="Enter budget amount" step="0.01" min="0">
        </div>

        <div class="budget-form-buttons"> <!--NEWWWWWWWW-->
          <button class="budget-btn cancel-btn" @click="cancelBudgetForm">Cancel </button>
            <button class="budget-btn" @click="addBudget">Set Budget</button>
      
          </div>
    </div>

            <!-- FOR EDITING BUDGET -->
            <div v-if="isEditingBudget" class="budget-form">
          <div class="form-group">
            <label for="editMonthYear">Month-Year:</label>
            <select id="editMonthYear" v-model="selectedMonthYear">
            <option v-for="month in availableMonths" :key="month" :value="month">{{ formatMonthYear(month) }}</option>
            </select>
          </div>
          <div class="form-group">
            <label for="editBudgetAmount">Budget Amount (₱):</label>
            <input type="number" id="editBudgetAmount" v-model="budgetAmount" placeholder="Enter budget amount" step="0.01" min="0" >
          </div>
          <div class="budget-form-buttons">
            <button class="budget-btn cancel-btn" @click="cancelBudgetForm">Cancel</button>
            <button class="budget-btn" @click="updateBudget">Update Budget</button>
          </div>
        </div>
</div>
</div>


  <!--ADDING EXPENSESSSS-->
    <div class="content-wrapper">
      <form @submit.prevent="handleSubmit" class="expense-form"> <!-- CLASS IS NEWWWWWWWWWWW-->
         <input type="hidden" v-model="action" />
         <input type="hidden" v-if="editId" v-model="editId" />


         <div class="form-group">
           <label>EXPENSE TYPE:</label>
           <select v-model="expenseType" required @change="checkExpenseType">
             <option value="Food">Food</option>
             <option value="Bill">Bill</option>
             <option value="Transportation">Transportation</option>
             <option value="Entertainment">Entertainment</option>
             <option value="Healthcare">Healthcare</option>
             <option value="Personalcare">Personal care</option>
             <option value="Shopping">Shopping</option> 
             <option value="Other">Other</option>
            </select>
         </div>

         <div v-if="expenseType === 'Other'" class="form-group">
           <label>Custom Expense Type:</label>
           <input type="text" v-model="customExpenseType" placeholder="Enter custom expense type" />
         </div>
 
         <div class="form-group">
           <label>ITEM NAME:</label>
           <input type="text" v-model="itemName" placeholder="Enter item name" required />
         </div>
 
         <div class="form-group">
           <label>ITEM PRICE:</label>
           <input type="number" v-model.number="itemPrice" placeholder="Enter item price" required step="0.01" />
         </div>
 
         <button class="btn" type="submit">{{ editId ? 'Save Changes' : 'Add Expense' }}</button>
         <div v-if="expenseSuccessMessage" class="expense-success-message" :class="{ hide: expenseHideMessage }">{{ expenseSuccessMessage }}</div>
      </form>

      </div>
      </div>

      <!--YOUR LIST OF EXPENSES-->
      <div class="expenses-container">
      <div class="expenses-section"> 
        <h3>Your Expenses</h3> 
         <div class="expenses-table"> 
          <table>
            <thead>
              <tr>
                <th>Expense Type</th>
                <th>Item Name</th>
                <th>Item Price</th>
                <th>Date</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="expense in expenses" :key="expense.id">
                <td>{{ expense.expense_type }}</td>
                <td>{{ expense.item_name }}</td>
                <td>{{ formatPHP(expense.item_price) }}</td>
                <td>{{ formatDate(expense.expense_date) }}</td>
                  <td class="actions">
                  <button @click="editExpense(expense)" class="edit-btn">Edit</button>
                  <button @click="deleteExpense(expense.id)" class="delete-btn">Delete</button>
                </td>
                </tr>
            </tbody>
          </table>
        </div>
       </div>
       
     <div class="total">
      Total: <strong>₱{{ totalAmount.toFixed(2) }}</strong> (≈ {{ formatUsd(convertPhpToUsd(totalAmount)) }} USD)
     </div>
    </div>
  </div>
 </template>
 
 <script>
import Navigation from "./navigation.vue"; 
 
 export default {
    props: {
    groupId: {
      type: [String, Number],
      required: true
    }
  }, //NEWWWWW
   components: { Navigation },
   data() {
     return {
       MemberName: '',
       expenseType: '',
       customExpenseType: '',
       itemName: '',
       itemPrice: '',
       successMessage: '',
       editId: null,
       action: 'add',
       expenses: [],
       currentGroup: null,
       currentUserRole: null,
       showInviteDialog: false
     };
   },
   computed: {
     totalAmount() {
       return this.expenses.reduce((sum, expense) => sum + expense.price, 0);
     }
   },
   
   async created() {
    try {
      // Use the prop if available, otherwise fetch groups
      if (this.groupId) {
        const response = await this.$axios.get(`/api/grp_expenses/${this.groupId}`);
        this.currentGroup = response.data.data;
        await this.fetchGroupData();
    }
  } catch (err) {
    console.error('Failed to load group data:', err);
    if (err.response?.status === 401) {
      this.$router.push('/login');
    } else if (err.response?.status === 404) {
      this.$router.push('/GC');
    }
  }
},


   methods: {
    checkExpenseType() {
       if (this.expenseType === "Other") this.customExpenseType = "";
     },
     handleSubmit() { /* Logic here */ },

    async fetchGroupData() {
      try {
        const response = await this.$axios.get('/api/grp_expenses/current');
        this.currentGroup = response.data.group;
        this.currentUserRole = response.data.role;
      } catch (err) {
        console.error("Failed to fetch group data:", err);
      }
    },
    
    async deleteGroup() {
      if (confirm("Are you sure you want to delete this group?")) {
        try {
          await this.$axios.delete(`/api/grp_expenses/${this.currentGroup.id}`);
          this.$router.push('/GC'); // or this.$router.push('/groups');
        } catch (err) {
          console.error("Failed to delete group:", err);
        }
      }
    }
  }
};
 </script>
 
 
 <style scoped>
 .main-layout {
   display: flex;
   flex-direction: column;
   width: 100%;
   gap: 20px;
   padding: 20px;
   box-sizing: border-box;
 }
 
 .top-row {
   margin-top: 100px;
   display: flex;
   gap: 20px;
   width: 100%;
 } 
 
 /* Budget Container Styles */
 .budget-container {
   width: 30%;
   background-color: #4a7c59; 
   padding: 25px;
   border-radius: 15px;
   border: 2px solid #2e4e38;
   box-shadow: 0 4px 8px rgba(0,0,0,0.2);
   height: auto;
   color: white;
 }
 
 .budget-header {
   flex-wrap: wrap;
   display: flex;
   align-items: center;
   margin-bottom: 25px;
 }
 
 .budget-header h3 {
   color: white;
   font-size: 1.5rem;
   margin-right: 16px;
 }
 
 .budget-content {
   width: 100%;
 }
 
 .add-budget-btn, .edit-budget-btn {
   background: #2a4935;
   color: white;
   border: none;
   padding: 10px 15px;
   border-radius: 8px;
   cursor: pointer;
   font-size: 0.9rem;
   transition: all 0.3s;
   box-shadow: 0 2px 4px rgba(0,0,0,0.1);
   max-width: 72px;
   font-size: 14px;
   margin-left: 5px;
 }
 
 
 .add-budget-btn:hover, .edit-budget-btn:hover {
   background: #12301f;
   transform: translateY(-2px);
 }
 
 .budget-display {
   flex-wrap: wrap;
   background: rgba(255, 255, 255, 0.1);
   padding: 20px;
   border-radius: 10px;
   margin-bottom: 20px;
 }
 
 .budget-info {
   display: flex;
   flex-direction: column;
   gap: 15px;
 }
 
 .budget-month-row, .budget-amount-row {
   flex-wrap: wrap;
   display: flex;
   justify-content: space-between;
   align-items: center;
 }
 
 .budget-label {
   font-weight: 500;
   font-size: 1rem;
 }
 .budget-month, .budget-amount {
   font-weight: 600;
   font-size: 1.1rem;
 }
 
 .budget-amount {
   color: #ffea00; 
 }
 
 .budget-form {
   display: flex;
   flex-direction: column;
   gap: 20px;
   background: rgba(255, 255, 255, 0.1);
   padding: 20px;
   border-radius: 10px;
 }
 
 .budget-form .form-group {
   margin: 0;
 }
 
 .budget-form label {
   color: white;
   margin-bottom: 8px;
 }
 
 .budget-form input, .budget-form select {
   width: 100%;
   padding: 12px;
   border-radius: 8px;
   border: 1px solid #ddd;
   background: white;
   color: #333;
 }
 
 .budget-form-buttons {
   display: flex;
   flex-wrap: wrap; /* allows buttons to wrap on small screens */
   gap: 12px;        /* space between buttons */
   justify-content: center; /* or 'center' if you prefer */
   box-sizing: border-box;
   margin-top: 20px;
   width: 100%;
 }
 
 .budget-btn {
   padding: 12px 0;
   background-color: #2a4935;
   color: white;
   border: none;
   border-radius: 8px;
   cursor: pointer;
   font-size: 1rem;
   transition: all 0.3s;
   flex: 1;
 }
 
 .budget-btn:hover {
   background-color: #12301f;
   transform: translateY(-2px);
 }
 
 .cancel-btn {
   background-color: #6c757d;
 }
 
 .cancel-btn:hover {
   background-color: #4b5256;
 }
 
 .budget-success-message {
   background-color: #d4edda;
   color: #155724;
   padding: 10px;
   border-radius: 4px;
   margin: 10px 0;
   text-align: center;
   transition: opacity 0.5s ease;
 }
 
 .expense-success-message {
   background-color: #d4edda;
   color: #155724;
   padding: 10px;
   border-radius: 4px;
   margin: 10px 0;
   text-align: center;
   transition: opacity 0.5s ease;
 } /* for expenses*/
 
 .budget-success-message.hide,
 .expense-success-message.hide {
   opacity: 0;
 }
 
 
 .uneditable-month {
   display: inline-block;
   padding: 8px 12px;
   background: rgba(255, 255, 255, 0.2);
   border-radius: 4px;
   color: #ffea00;
   font-weight: bold;
 }
  
  .content-wrapper {
   align-content: center;
   width: 70%;
   background-color: #85cf9d;
   border: 2px solid #365c42;
   padding: 20px;
   border-radius: 15px;
   box-shadow: 0 4px 8px rgba(0,0,0,0.1);
   height: auto;
 }
 
 .expenses-container {
   max-width: 100%; 
   margin: 0 auto;
   box-sizing: border-box;
   width: 100%;
   background-color: white;
   padding: 20px;
   border-radius: 15px;
   border: 2px solid #85cf9d;
   box-shadow: 0 4px 8px rgba(0,0,0,0.1);
 }
  
 
  .expense-form {
   text-align: center;
   width: 100%; 
 }  
  
 
 .expenses-section {
   margin-top: 10px; 
 }
 
 .expenses-section h3 {
   margin-top: 10px;
   margin-bottom: 25px; 
   color: #333;
   font-size: 1.5rem; 
   padding-bottom: 10px;
   border-bottom: 2px solid #eee; 
 } 
 
 .expenses-table {
   overflow-x: auto;
   margin: 30px 0; 
 }  
 
 table {
   width: 100%;
   border-collapse: separate; 
   border-spacing: 0 10px; 
   margin-bottom: 30px; 
 }  
 
 th, td {
   padding: 6px 20px; 
   text-align: center;
   border-bottom: 2px solid #ddd;
   color: #333;
 } 
 
 th {
   background-color: #f8f9fa;
   font-weight: 600;
   font-size: 1rem; 
   padding: 12px 20px; 
 } 
 
 tr {
   background-color: white;
   box-shadow: 0 2px 4px rgba(0,0,0,0.05); 
   margin-bottom: 15px; 
 }
 
 tr:hover {
   background-color: #f5f5f5;
   transform: translateY(-2px); 
   box-shadow: 0 4px 8px rgba(0,0,0,0.1); 
   transition: all 0.2s ease; 
 } 
 
 .actions {
   display: flex;
   gap: 10px;
   justify-content: center;
 } 
 
 .edit-btn, .delete-btn {
   padding: 8px 15px;
   font-size: 0.9rem;
   border-radius: 4px;
   cursor: pointer;
   border: none;
   color: white;
   font-weight: 500;
   position: relative;
   overflow: hidden;
   transition: all 0.3s ease;
   box-shadow: 0 2px 5px rgba(0,0,0,0.1);
 }
 
 .edit-btn {
   background-color: #2196F3;
 }
 
 
 .delete-btn {
   background-color: #f44336;
 }
 
 
 .edit-btn:hover {
   background-color: #1976D2;
   transform: translateY(-2px);
   box-shadow: 0 4px 8px rgba(33, 150, 243, 0.3);
   
 
   &::after {
     content: '';
     position: absolute;
     top: 50%;
     left: 50%;
     width: 5px;
     height: 5px;
     background: rgba(255, 255, 255, 0.5);
     opacity: 0;
     border-radius: 100%;
     transform: scale(1, 1) translate(-50%);
     transform-origin: 50% 50%;
   }
   
   &:hover::after {
     animation: ripple 0.6s ease-out;
   }
 }
 
 .delete-btn:hover {
   background-color: #d32f2f;
   transform: translateY(-2px);
   box-shadow: 0 4px 8px rgba(244, 67, 54, 0.3);
   
 
   animation: pulse 0.5s ease-in-out;
 }
 
 
 @keyframes ripple {
   0% {
     transform: scale(0, 0);
     opacity: 0.5;
   }
   100% {
     transform: scale(20, 20);
     opacity: 0;
   }
 }
 
 
 @keyframes pulse {
   0% {
     transform: translateY(-2px) scale(1);
   }
   50% {
     transform: translateY(-2px) scale(1.05);
   }
   100% {
     transform: translateY(-2px) scale(1);
   }
 }
 
 
 .edit-btn:active, .delete-btn:active {
   transform: translateY(0);
   box-shadow: 0 1px 3px rgba(0,0,0,0.2);
 }
 
 .total {
      font-size: 20px;
      font-weight: bold;
      color: #333;
      padding: 20px;
      background-color: white;
      box-sizing: border-box;
      box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
      text-align: center;
      max-width: 1300px;
      width: 100%;
      position: relative; 
      bottom: 0;   
 }
 
  
  .form-group {
      margin-bottom: 20px;
      margin-top: 20px;
  }
  
  label {
      margin-left: 10px;
      text-align: left;
      display: block;
      margin-bottom: 5px;
      font-size: 17px;
      color: black;
  }
  
  input[type="text"],
  input[type="number"],
  select {
      width: 100%;
      max-width: 800px;
      padding: 10px;
      font-size: 16px;
      border-radius: 10px;
      border: 3px solid #ddd;
      border-color: #2a4935;
      box-sizing: border-box;
      min-height: 35px;
  }
  
  .btn {
      padding: 12px 50px;
      background-color: #2a4935;
      box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
      color: white;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      font-size: 15px;
      transition: background-color 0.3s, color 0.3s; /* Smooth effect */
 }
 
 .btn:hover {
     background-color: #12301f; /* Change to any color you want */
     color: white; /* Text color on hover */
     transform: translateY(-2px);
 }
 
 .btn1 {
     padding: 12px 50px;
     background-color: white;
     box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
     color: black;
     border: none;
     border-radius: 12px;
     cursor: pointer;
     font-size: 15px;
     margin-top: 10px; /* Add vertical spacing instead */
     margin-left: 430px;
     transition: background-color 0.3s, color 0.3s;
 }
 
 .btn1:hover {
     background-color: rgb(26, 25, 25); /* Change to any color you want */
     color: white; /* Text color on hover */
 }
 
 /* RESPONSIVE DESIGN */
 
 @media screen and (max-width: 1000px) {
     .container {
         margin: 20px 0px 0px 30px;
         padding: 15px;
         height: 500px;
     }
 
     .expense-form{
       margin-top: 10px;
       margin-bottom: 10px;
     }
 
     .add-budget-btn, .edit-budget-btn {
       margin-bottom: 10px;
     }
 
     .content-wrapper {
         padding: 0px;
     }
 
     input[type="text"],
     input[type="number"],
     select {
         width: 90%;
         max-width: 600px;
         font-size: 15px;
     }
 
     .h3{
       font-size: 20px;
       font-weight: bold;
       margin-top: 30px;
       color: white;
     }
 
     .expenses-container{
         width: 100%;
         max-width: 1000px;
     }
 }
 
 </style>
