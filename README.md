# myhr
#this is my first systems creating something new
import React, { useEffect, useState, createContext, useContext } from 'react'
import { useCompanyContext } from './CompanyContext'
// Types
interface Employee {
  id: string
  companyId: string
  firstName: string
  lastName: string
  email: string
  phone: string
  department: string
  position: string
  hireDate: string
  status: 'active' | 'on leave' | 'terminated'
}
interface Attendance {
  id: string
  companyId: string
  employeeId: string
  date: string
  status: 'present' | 'absent' | 'late'
}
interface LeaveRequest {
  id: string
  companyId: string
  employeeId: string
  type: string
  startDate: string
  endDate: string
  reason: string
  status: 'pending' | 'approved' | 'rejected'
}
interface PerformanceReview {
  id: string
  companyId: string
  employeeId: string
  reviewerId: string
  reviewDate: string
  performanceScore: number
  strengths: string
  areasForImprovement: string
  goals: string
  comments: string
}
// New interfaces for payroll functionality
interface Allowance {
  type: string
  amount: number
}
interface Deduction {
  type: string
  amount: number
}
interface SalaryStructure {
  id: string
  companyId: string
  employeeId: string
  baseSalary: number
  allowances: Allowance[]
  deductions: Deduction[]
  effectiveDate: string
}
interface PayrollRecord {
  id: string
  companyId: string
  employeeId: string
  periodStart: string
  periodEnd: string
  baseSalary: number
  allowances: Allowance[]
  deductions: Deduction[]
  tax: number
  netSalary: number
  paymentDate: string
  paymentStatus: 'pending' | 'paid' | 'failed'
}
interface HRContextType {
  employees: Employee[]
  attendance: Attendance[]
  leaveRequests: LeaveRequest[]
  performanceReviews: PerformanceReview[]
  salaryStructures: SalaryStructure[]
  payrollRecords: PayrollRecord[]
  addEmployee: (employee: Employee) => void
  updateEmployee: (employee: Employee) => void
  deleteEmployee: (id: string) => void
  recordAttendance: (attendance: Attendance) => void
  addLeaveRequest: (request: LeaveRequest) => void
  updateLeaveRequest: (request: Partial<LeaveRequest>) => void
  addPerformanceReview: (review: PerformanceReview) => void
  loadCompanyData: (companyId: string) => void
  addSalaryStructure: (salary: SalaryStructure) => void
  updateSalaryStructure: (salary: SalaryStructure) => void
  deleteSalaryStructure: (id: string) => void
  addPayrollRecord: (payroll: PayrollRecord) => void
  updatePayrollRecord: (payroll: Partial<PayrollRecord>) => void
  calculateNetSalary: (
    employeeId: string,
    periodStart: string,
    periodEnd: string,
  ) => PayrollRecord | null
}
// Local storage keys
const EMPLOYEES_STORAGE_KEY = 'easy_co_employees'
const ATTENDANCE_STORAGE_KEY = 'easy_co_attendance'
const LEAVE_REQUESTS_STORAGE_KEY = 'easy_co_leave_requests'
const PERFORMANCE_REVIEWS_STORAGE_KEY = 'easy_co_performance_reviews'
const SALARY_STRUCTURES_STORAGE_KEY = 'easy_co_salary_structures'
const PAYROLL_RECORDS_STORAGE_KEY = 'easy_co_payroll_records'
// Initial mock data generator
const generateInitialData = (companyId: string) => {
  const initialEmployees: Employee[] = [
    {
      id: '1',
      companyId,
      firstName: 'John',
      lastName: 'Doe',
      email: 'john.doe@company.com',
      phone: '(555) 123-4567',
      department: 'Engineering',
      position: 'Senior Developer',
      hireDate: '2020-01-15',
      status: 'active',
    },
    {
      id: '2',
      companyId,
      firstName: 'Jane',
      lastName: 'Smith',
      email: 'jane.smith@company.com',
      phone: '(555) 987-6543',
      department: 'Marketing',
      position: 'Marketing Manager',
      hireDate: '2019-06-22',
      status: 'active',
    },
    {
      id: '3',
      companyId,
      firstName: 'Robert',
      lastName: 'Johnson',
      email: 'robert.johnson@company.com',
      phone: '(555) 456-7890',
      department: 'Finance',
      position: 'Financial Analyst',
      hireDate: '2021-03-10',
      status: 'on leave',
    },
    {
      id: '4',
      companyId,
      firstName: 'Sarah',
      lastName: 'Williams',
      email: 'sarah.williams@company.com',
      phone: '(555) 234-5678',
      department: 'HR',
      position: 'HR Specialist',
      hireDate: '2018-11-05',
      status: 'active',
    },
    {
      id: '5',
      companyId,
      firstName: 'Michael',
      lastName: 'Brown',
      email: 'michael.brown@company.com',
      phone: '(555) 876-5432',
      department: 'Sales',
      position: 'Sales Representative',
      hireDate: '2022-02-18',
      status: 'active',
    },
  ]
  const initialAttendance: Attendance[] = [
    {
      id: '1',
      companyId,
      employeeId: '1',
      date: new Date().toISOString().split('T')[0],
      status: 'present',
    },
    {
      id: '2',
      companyId,
      employeeId: '2',
      date: new Date().toISOString().split('T')[0],
      status: 'present',
    },
    {
      id: '3',
      companyId,
      employeeId: '4',
      date: new Date().toISOString().split('T')[0],
      status: 'present',
    },
    {
      id: '4',
      companyId,
      employeeId: '5',
      date: new Date().toISOString().split('T')[0],
      status: 'late',
    },
  ]
  const initialLeaveRequests: LeaveRequest[] = [
    {
      id: '1',
      companyId,
      employeeId: '3',
      type: 'sick',
      startDate: '2023-06-10',
      endDate: '2023-06-15',
      reason: 'Medical appointment and recovery',
      status: 'approved',
    },
    {
      id: '2',
      companyId,
      employeeId: '2',
      type: 'vacation',
      startDate: '2023-07-01',
      endDate: '2023-07-10',
      reason: 'Family vacation',
      status: 'pending',
    },
    {
      id: '3',
      companyId,
      employeeId: '5',
      type: 'personal',
      startDate: '2023-06-20',
      endDate: '2023-06-21',
      reason: 'Personal matters',
      status: 'pending',
    },
  ]
  const initialPerformanceReviews: PerformanceReview[] = [
    {
      id: '1',
      companyId,
      employeeId: '1',
      reviewerId: '4',
      reviewDate: '2023-01-15',
      performanceScore: 4,
      strengths:
        'Excellent technical skills, great team player, delivers high-quality work consistently.',
      areasForImprovement:
        'Could improve documentation practices and communication with non-technical team members.',
      goals:
        'Lead a major project in the next quarter, mentor junior developers.',
      comments:
        'John has been an outstanding contributor this year. His technical expertise has been crucial for several key projects.',
    },
    {
      id: '2',
      companyId,
      employeeId: '2',
      reviewerId: '4',
      reviewDate: '2023-02-10',
      performanceScore: 5,
      strengths:
        'Outstanding leadership, excellent communication skills, innovative marketing strategies.',
      areasForImprovement: 'Could delegate more tasks to team members.',
      goals:
        'Develop and implement a comprehensive digital marketing strategy, increase brand awareness by 20%.',
      comments:
        'Jane has exceeded expectations in her role as Marketing Manager. Her campaigns have significantly increased our market presence.',
    },
  ]
  // Initial salary structures
  const initialSalaryStructures: SalaryStructure[] = [
    {
      id: '1',
      companyId,
      employeeId: '1',
      baseSalary: 85000,
      allowances: [
        {
          type: 'Housing',
          amount: 1000,
        },
        {
          type: 'Transportation',
          amount: 500,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 425,
        },
      ],
      effectiveDate: '2023-01-01',
    },
    {
      id: '2',
      companyId,
      employeeId: '2',
      baseSalary: 92000,
      allowances: [
        {
          type: 'Housing',
          amount: 1200,
        },
        {
          type: 'Transportation',
          amount: 500,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 460,
        },
      ],
      effectiveDate: '2023-01-01',
    },
    {
      id: '3',
      companyId,
      employeeId: '3',
      baseSalary: 75000,
      allowances: [
        {
          type: 'Housing',
          amount: 800,
        },
        {
          type: 'Transportation',
          amount: 400,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 375,
        },
      ],
      effectiveDate: '2023-01-01',
    },
    {
      id: '4',
      companyId,
      employeeId: '4',
      baseSalary: 78000,
      allowances: [
        {
          type: 'Housing',
          amount: 900,
        },
        {
          type: 'Transportation',
          amount: 450,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 390,
        },
      ],
      effectiveDate: '2023-01-01',
    },
    {
      id: '5',
      companyId,
      employeeId: '5',
      baseSalary: 72000,
      allowances: [
        {
          type: 'Housing',
          amount: 800,
        },
        {
          type: 'Transportation',
          amount: 400,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 360,
        },
      ],
      effectiveDate: '2023-01-01',
    },
  ]
  // Initial payroll records
  const initialPayrollRecords: PayrollRecord[] = [
    {
      id: '1',
      companyId,
      employeeId: '1',
      periodStart: '2023-05-01',
      periodEnd: '2023-05-31',
      baseSalary: 7083.33,
      allowances: [
        {
          type: 'Housing',
          amount: 1000,
        },
        {
          type: 'Transportation',
          amount: 500,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 425,
        },
      ],
      tax: 1771.67,
      netSalary: 6086.66,
      paymentDate: '2023-06-01',
      paymentStatus: 'paid',
    },
    {
      id: '2',
      companyId,
      employeeId: '2',
      periodStart: '2023-05-01',
      periodEnd: '2023-05-31',
      baseSalary: 7666.67,
      allowances: [
        {
          type: 'Housing',
          amount: 1200,
        },
        {
          type: 'Transportation',
          amount: 500,
        },
      ],
      deductions: [
        {
          type: 'Health Insurance',
          amount: 300,
        },
        {
          type: 'Retirement Plan',
          amount: 460,
        },
      ],
      tax: 1916.67,
      netSalary: 6690.0,
      paymentDate: '2023-06-01',
      paymentStatus: 'paid',
    },
  ]
  return {
    initialEmployees,
    initialAttendance,
    initialLeaveRequests,
    initialPerformanceReviews,
    initialSalaryStructures,
    initialPayrollRecords,
  }
}
// Create context
const HRContext = createContext<HRContextType | undefined>(undefined)
export const HRProvider: React.FC<{
  children: React.ReactNode
}> = ({ children }) => {
  const { currentCompany } = useCompanyContext()
  const companyId = currentCompany?.id || 'company1'
  // Load data from localStorage or generate initial data
  const loadFromStorage = <
    T extends {
      companyId: string
    },
  >(
    key: string,
    initialData: T[],
  ): T[] => {
    const storedData = localStorage.getItem(key)
    if (storedData) {
      const allData = JSON.parse(storedData) as T[]
      // Filter by company ID
      return allData.filter((item) => item.companyId === companyId)
    }
    // If no data in storage, use initial data
    return initialData
  }
  // Generate initial data for the current company
  const {
    initialEmployees,
    initialAttendance,
    initialLeaveRequests,
    initialPerformanceReviews,
    initialSalaryStructures,
    initialPayrollRecords,
  } = generateInitialData(companyId)
  // State for HR data
  const [employees, setEmployees] = useState<Employee[]>(() =>
    loadFromStorage(EMPLOYEES_STORAGE_KEY, initialEmployees),
  )
  const [attendance, setAttendance] = useState<Attendance[]>(() =>
    loadFromStorage(ATTENDANCE_STORAGE_KEY, initialAttendance),
  )
  const [leaveRequests, setLeaveRequests] = useState<LeaveRequest[]>(() =>
    loadFromStorage(LEAVE_REQUESTS_STORAGE_KEY, initialLeaveRequests),
  )
  const [performanceReviews, setPerformanceReviews] = useState<
    PerformanceReview[]
  >(() =>
    loadFromStorage(PERFORMANCE_REVIEWS_STORAGE_KEY, initialPerformanceReviews),
  )
  const [salaryStructures, setSalaryStructures] = useState<SalaryStructure[]>(
    () =>
      loadFromStorage(SALARY_STRUCTURES_STORAGE_KEY, initialSalaryStructures),
  )
  const [payrollRecords, setPayrollRecords] = useState<PayrollRecord[]>(() =>
    loadFromStorage(PAYROLL_RECORDS_STORAGE_KEY, initialPayrollRecords),
  )
  // Function to load data for a specific company
  const loadCompanyData = (companyId: string) => {
    const {
      initialEmployees,
      initialAttendance,
      initialLeaveRequests,
      initialPerformanceReviews,
      initialSalaryStructures,
      initialPayrollRecords,
    } = generateInitialData(companyId)
    setEmployees(loadFromStorage(EMPLOYEES_STORAGE_KEY, initialEmployees))
    setAttendance(loadFromStorage(ATTENDANCE_STORAGE_KEY, initialAttendance))
    setLeaveRequests(
      loadFromStorage(LEAVE_REQUESTS_STORAGE_KEY, initialLeaveRequests),
    )
    setPerformanceReviews(
      loadFromStorage(
        PERFORMANCE_REVIEWS_STORAGE_KEY,
        initialPerformanceReviews,
      ),
    )
    setSalaryStructures(
      loadFromStorage(SALARY_STRUCTURES_STORAGE_KEY, initialSalaryStructures),
    )
    setPayrollRecords(
      loadFromStorage(PAYROLL_RECORDS_STORAGE_KEY, initialPayrollRecords),
    )
  }
  // Load data when company changes
  useEffect(() => {
    if (currentCompany) {
      loadCompanyData(currentCompany.id)
    }
  }, [currentCompany])
  // Save data to localStorage whenever it changes
  const saveToStorage = <T,>(key: string, data: T) => {
    const existingData = localStorage.getItem(key)
    if (existingData) {
      // If we have existing data, we need to merge with data from other companies
      const allData = JSON.parse(existingData)
      const filteredData = Array.isArray(allData)
        ? allData.filter((item: any) => item.companyId !== companyId)
        : []
      // Combine data from other companies with current company data
      const combinedData = [...filteredData, ...data]
      localStorage.setItem(key, JSON.stringify(combinedData))
    } else {
      // No existing data, just save current data
      localStorage.setItem(key, JSON.stringify(data))
    }
  }
  useEffect(() => {
    saveToStorage(EMPLOYEES_STORAGE_KEY, employees)
  }, [employees, companyId])
  useEffect(() => {
    saveToStorage(ATTENDANCE_STORAGE_KEY, attendance)
  }, [attendance, companyId])
  useEffect(() => {
    saveToStorage(LEAVE_REQUESTS_STORAGE_KEY, leaveRequests)
  }, [leaveRequests, companyId])
  useEffect(() => {
    saveToStorage(PERFORMANCE_REVIEWS_STORAGE_KEY, performanceReviews)
  }, [performanceReviews, companyId])
  useEffect(() => {
    saveToStorage(SALARY_STRUCTURES_STORAGE_KEY, salaryStructures)
  }, [salaryStructures, companyId])
  useEffect(() => {
    saveToStorage(PAYROLL_RECORDS_STORAGE_KEY, payrollRecords)
  }, [payrollRecords, companyId])
  // CRUD operations for employees
  const addEmployee = (employee: Employee) => {
    const employeeWithCompany = {
      ...employee,
      companyId,
    }
    setEmployees([...employees, employeeWithCompany])
  }
  const updateEmployee = (updatedEmployee: Employee) => {
    setEmployees(
      employees.map((emp) =>
        emp.id === updatedEmployee.id
          ? {
              ...updatedEmployee,
              companyId,
            }
          : emp,
      ),
    )
  }
  const deleteEmployee = (id: string) => {
    setEmployees(employees.filter((emp) => emp.id !== id))
  }
  // Attendance operations
  const recordAttendance = (newAttendance: Attendance) => {
    // Add company ID and unique ID if not provided
    const attendanceWithCompany = {
      ...newAttendance,
      companyId,
      id: newAttendance.id || Date.now().toString(),
    }
    // Remove existing attendance record for the employee on the same date if exists
    const filteredAttendance = attendance.filter(
      (a) =>
        !(
          a.employeeId === attendanceWithCompany.employeeId &&
          a.date === attendanceWithCompany.date
        ),
    )
    setAttendance([...filteredAttendance, attendanceWithCompany])
  }
  // Leave request operations
  const addLeaveRequest = (request: LeaveRequest) => {
    const requestWithCompany = {
      ...request,
      companyId,
    }
    setLeaveRequests([...leaveRequests, requestWithCompany])
  }
  const updateLeaveRequest = (updatedRequest: Partial<LeaveRequest>) => {
    setLeaveRequests(
      leaveRequests.map((req) =>
        req.id === updatedRequest.id
          ? {
              ...req,
              ...updatedRequest,
              companyId,
            }
          : req,
      ),
    )
  }
  // Performance review operations
  const addPerformanceReview = (review: PerformanceReview) => {
    const reviewWithCompany = {
      ...review,
      companyId,
    }
    setPerformanceReviews([...performanceReviews, reviewWithCompany])
  }
  // Salary structure operations
  const addSalaryStructure = (salary: SalaryStructure) => {
    const salaryWithCompany = {
      ...salary,
      companyId,
    }
    setSalaryStructures([...salaryStructures, salaryWithCompany])
  }
  const updateSalaryStructure = (updatedSalary: SalaryStructure) => {
    setSalaryStructures(
      salaryStructures.map((sal) =>
        sal.id === updatedSalary.id
          ? {
              ...updatedSalary,
              companyId,
            }
          : sal,
      ),
    )
  }
  const deleteSalaryStructure = (id: string) => {
    setSalaryStructures(salaryStructures.filter((sal) => sal.id !== id))
  }
  // Payroll record operations
  const addPayrollRecord = (payroll: PayrollRecord) => {
    const payrollWithCompany = {
      ...payroll,
      companyId,
    }
    setPayrollRecords([...payrollRecords, payrollWithCompany])
  }
  const updatePayrollRecord = (updatedPayroll: Partial<PayrollRecord>) => {
    setPayrollRecords(
      payrollRecords.map((rec) =>
        rec.id === updatedPayroll.id
          ? {
              ...rec,
              ...updatedPayroll,
              companyId,
            }
          : rec,
      ),
    )
  }
  // Utility function to calculate net salary for an employee
  const calculateNetSalary = (
    employeeId: string,
    periodStart: string,
    periodEnd: string,
  ): PayrollRecord | null => {
    // Find the employee's salary structure
    const salaryStructure = salaryStructures.find(
      (sal) => sal.employeeId === employeeId,
    )
    if (!salaryStructure) {
      return null
    }
    // Calculate monthly base salary (assuming annual salary)
    const monthlyBaseSalary = salaryStructure.baseSalary / 12
    // Calculate total allowances
    const totalAllowances = salaryStructure.allowances.reduce(
      (sum, allowance) => sum + allowance.amount,
      0,
    )
    // Calculate total deductions
    const totalDeductions = salaryStructure.deductions.reduce(
      (sum, deduction) => sum + deduction.amount,
      0,
    )
    // Calculate tax (simplified - assume 25% of base salary)
    const tax = monthlyBaseSalary * 0.25
    // Calculate net salary
    const netSalary =
      monthlyBaseSalary + totalAllowances - totalDeductions - tax
    // Create payroll record
    const payrollRecord: PayrollRecord = {
      id: Date.now().toString(),
      companyId,
      employeeId,
      periodStart,
      periodEnd,
      baseSalary: monthlyBaseSalary,
      allowances: [...salaryStructure.allowances],
      deductions: [...salaryStructure.deductions],
      tax,
      netSalary,
      paymentDate: new Date().toISOString().split('T')[0],
      paymentStatus: 'pending',
    }
    return payrollRecord
  }
  return (
    <HRContext.Provider
      value={{
        employees,
        attendance,
        leaveRequests,
        performanceReviews,
        salaryStructures,
        payrollRecords,
        addEmployee,
        updateEmployee,
        deleteEmployee,
        recordAttendance,
        addLeaveRequest,
        updateLeaveRequest,
        addPerformanceReview,
        loadCompanyData,
        addSalaryStructure,
        updateSalaryStructure,
        deleteSalaryStructure,
        addPayrollRecord,
        updatePayrollRecord,
        calculateNetSalary,
      }}
    >
      {children}
    </HRContext.Provider>
  )
}
export const useHRContext = () => {
  const context = useContext(HRContext)
  if (context === undefined) {
    throw new Error('useHRContext must be used within an HRProvider')
  }
  return context
}

