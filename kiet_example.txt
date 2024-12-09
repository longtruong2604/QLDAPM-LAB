import { DatePicker, Space, Tag } from 'antd'
import { CustomTable } from 'components'
import dayjs from 'dayjs'
import customParseFormat from 'dayjs/plugin/customParseFormat'
import { useEffect, useState } from 'react'
import './ManageMenu.scss'
import OrderedDishesTable from './OrderedDishesTable'
import { formatCurrency } from 'components/CommonFunction/formatCurrency'
import { useLocation } from 'react-router-dom'
import { MOCK_USER_ORDER_HISTORY_DATA } from 'pages/Management/ManageTable/MOCK_DATA'

dayjs.extend(customParseFormat)
const { RangePicker } = DatePicker

const ManageAccount = () => {
  // useNavigate

  const [data, setData] = useState([])
  const [filteredData, setFilteredData] = useState([])
  const location = useLocation()
  const [isLoading, setLoading] = useState(true)
  const [searchText, setSearchText] = useState('')
  const [searchedColumn, setSearchedColumn] = useState('')

  const handleDateChange = (date) => {
    if (date) {
      const start = date[0]
      const end = date[1]
      setFilteredData(
        data.filter((item) => {
          const date = new Date(item.date)
          return date >= start && date <= end
        })
      )
    } else {
      setFilteredData(data)
    }
  }

  const renderOrderedDishes = (record) => {
    const renderData = record.items.map((item) => ({
      ...item,
      key: item.dishId,
    }))
    return <OrderedDishesTable data={renderData} />
  }

  const columns = [
    {
      title: 'STT',
      key: 'stt',
      width: 50,
      render: (_, __, index) => index + 1,
    },
    {
      title: 'Order ID',
      dataIndex: 'orderId',
      key: 'orderId',
      width: 150,
      isSearched: true,
    },
    {
      title: 'Date',
      dataIndex: 'date',
      key: 'date',
      width: 100,
      render: (date) => new Date(date).toLocaleDateString('en-GB'),
      sorter: (a, b) => new Date(a.date) - new Date(b.date),
      isSearched: false,
    },
    {
      title: 'Số lượng',
      dataIndex: 'items',
      key: 'quantity',
      width: 100,
      render: (items) => items.reduce((acc, cur) => acc + cur.quantity, 0),
      isSearched: false,
    },
    {
      title: 'Total Amount (VND)',
      dataIndex: 'totalAmount',
      key: 'totalAmount',
      width: 150,
      render: (amount) => formatCurrency(amount),
      sorter: (a, b) => a.totalAmount - b.totalAmount,
      isSearched: false,
    },
    {
      title: 'Payment Method',
      dataIndex: 'paymentMethod',
      key: 'paymentMethod',
      width: 150,
      filters: [
        { text: 'Cash', value: 'Cash' },
        { text: 'Mobile Payment', value: 'Mobile Payment' },
        { text: 'Credit Card', value: 'Credit Card' },
      ],
      onFilter: (value, record) => record.paymentMethod === value,
      isSearched: false,
    },
    {
      title: 'Status',
      dataIndex: 'status',
      key: 'status',
      width: 120,
      filters: [
        { text: 'Completed', value: 'Completed' },
        { text: 'Pending', value: 'Pending' },
        { text: 'Cancelled', value: 'Cancelled' },
      ],
      onFilter: (value, record) => record.status === value,
      render: (status) => (
        <Tag color={status === 'Completed' ? 'green' : status === 'Cancelled' ? 'red' : 'orange'}>
          {status}
        </Tag>
      ),
      isSearched: false,
    },
  ]

  useEffect(() => {
    const simulateRequest = new Promise((resolve) => {
      setTimeout(() => resolve(MOCK_USER_ORDER_HISTORY_DATA), 1000) // Delay for 1 second to simulate a real request
    })

    simulateRequest
      .then((response) => {
        const loadedData = response.data.map((item, index) => ({
          ...item,
          key: index,
        }))
        setData(loadedData)
        setFilteredData(loadedData)
        setLoading(false)
      })
      .catch((error) => {
        console.error(error.message)
      })
  }, [location.pathname])

  return (
    <div className="ManageMenu">
      <Space className="manage-menu-filter">
        <div>
          <p>Lọc theo:</p>
          <div className="range-picker-container">
            <RangePicker
              onChange={handleDateChange}
              format="DD/MM/YYYY"
              placeholder={['Ngày bắt đầu', 'Ngày kết thúc']}
            />
          </div>
        </div>
      </Space>
      <CustomTable
        columns={columns}
        data={filteredData}
        isLoading={isLoading}
        searchText={searchText}
        searchedColumn={searchedColumn}
        setSearchText={setSearchText}
        setSearchedColumn={setSearchedColumn}
        expandedRowRender={renderOrderedDishes}
      />
    </div>
  )
}

export default ManageAccount
