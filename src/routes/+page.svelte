<script lang="ts">
  // Image paths for Peter and Sterling
  const peterImage = '/PeterSmiles.png';
  const sterlingImage = '/Sterling.png';
  
  // Handle image error with proper typing
  function handleImageError(event: Event) {
    const target = event.target as HTMLImageElement;
    if (target && target.nextElementSibling) {
      target.style.display = 'none';
      (target.nextElementSibling as HTMLElement).style.display = 'flex';
    }
  }
  
  // Handle LAPSH value change
  function handleLapshChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lapsh-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€187.00' : '€0.00';
      updateTotal();
      handleLsecChange();
    }
  }
  
  // Handle LSAS value change
  function handleLsasChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lsas-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€187.00' : '€0.00';
      updateTotal();
      handleLsecChange();
    }
  }
  
  // Handle LASH value change
  function handleLashChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lash-value');
    if (valueCell) {
      let baseValue = 0;
      
      // Get base tier value
      switch(target.value) {
        case 'TIER1':
          baseValue = 184;
          break;
        case 'TIER2':
          baseValue = 248;
          break;
        case 'TIER3':
          baseValue = 423;
          break;
        default:
          baseValue = 0;
      }
      
      // Get weight from shipment details
      const weightInput = document.getElementById('weight') as HTMLInputElement;
      const weight = weightInput ? parseFloat(weightInput.value) || 0 : 0;
      
      // Calculate additional weight charges (1.05 * excess kg over 22kg)
      let additionalWeight = 0;
      if (weight > 22) {
        additionalWeight = 1.05 * (weight - 22);
      }
      
      const totalValue = baseValue + additionalWeight;
      valueCell.textContent = totalValue > 0 ? `€${totalValue.toFixed(2)}` : '€0.00';
      updateTotal();
      handleLsecChange();
    }
  }
  
  // Handle GPS value change
  function handleGpsChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('gps-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€114.00' : '€0.00';
      
      // If GPS is set to YES, set LGPSL to NO
      if (target.value === 'YES') {
        const lgpslSelect = document.getElementById('lgpsl-select') as HTMLSelectElement;
        const lgpslValueCell = document.getElementById('lgpsl-value');
        if (lgpslSelect && lgpslValueCell) {
          lgpslSelect.value = 'NO';
          lgpslValueCell.textContent = '€0.00';
        }
      }
      
      updateTotal();
    }
  }
  
  // Handle LCOL value change
  function handleLcolChange() {
    const regionSelect = document.getElementById('lcol-region') as HTMLSelectElement;
    const milesInput = document.getElementById('lcol-miles') as HTMLInputElement;
    const valueCell = document.getElementById('lcol-value');
    
    if (valueCell && regionSelect && milesInput) {
      const region = regionSelect.value;
      const miles = parseFloat(milesInput.value) || 0;
      let value = 0;
      
      // First 15 miles are free for all regions
      if (miles > 15) {
        const chargableMiles = miles - 15;
        
        switch(region) {
          case 'US':
            value = chargableMiles * 2.36; // €2.36 per mile after 15 miles
            break;
          case 'EU-UK':
            value = chargableMiles * 3.70; // €3.70 per mile after 15 miles
            break;
          case 'ROW':
            value = chargableMiles * 4.78; // €4.78 per mile after 15 miles
            break;
          case 'JP':
            value = chargableMiles * 2.10; // €2.10 per mile after 15 miles
            break;
          default:
            value = 0;
        }
      }
      
      valueCell.textContent = value > 0 ? `€${value.toFixed(2)}` : '€0.00';
      updateTotal();
      // Trigger FS recalculation since LCOL affects it
      handleFsChange();
      // Trigger LSEC recalculation since LCOL affects it
      handleLsecChange();
    }
  }
  
  // Update running total
  function updateTotal() {
    let total = 0;
    
    // Get all value cells and sum them up
    const valueCells = document.querySelectorAll('td[id$="-value"]');
    valueCells.forEach(cell => {
      const text = cell.textContent || '';
      const value = parseFloat(text.replace('€', '').replace(',', ''));
      if (!isNaN(value)) {
        total += value;
      }
    });
    
    const totalCell = document.getElementById('running-total');
    if (totalCell) {
      totalCell.textContent = `€${total.toFixed(2)}`;
    }
  }
  
  // Handle weight change - recalculate LASH if tier is selected
  function handleWeightChange() {
    const lashSelect = document.getElementById('lash-select') as HTMLSelectElement;
    if (lashSelect && lashSelect.value !== 'NONE') {
      // Create a synthetic change event to trigger LASH recalculation
      const changeEvent = new Event('change');
      Object.defineProperty(changeEvent, 'target', { value: lashSelect });
      handleLashChange(changeEvent);
    }
    
    // Also trigger ADD recalculation since it depends on weight
    handleAddChange();
    
    // Also trigger LDRR recalculation since it depends on weight
    handleLdrrChange();
    
    // Also trigger LLW recalculation since it depends on weight
    handleLlwChange();
  }
  
  // Handle OOH value change
  function handleOohChange() {
    const pickupInput = document.getElementById('pickup-date') as HTMLInputElement;
    const valueCell = document.getElementById('ooh-value');
    
    if (valueCell && pickupInput && pickupInput.value) {
      const pickupDate = new Date(pickupInput.value);
      const dayOfWeek = pickupDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      const hour = pickupDate.getHours();
      
      let isOutOfHours = false;
      
      // Check if it's Monday-Friday (1-5) and between 5pm-8AM
      if (dayOfWeek >= 1 && dayOfWeek <= 5) {
        // Out of hours: 5pm (17:00) to 11:59pm OR 12:00am to 8am (08:00)
        if (hour >= 17 || hour < 8) {
          isOutOfHours = true;
        }
      }
      
      valueCell.textContent = isOutOfHours ? '€83.00' : '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation since OOH affects it
      handleLrdasChange();
      // Trigger FS recalculation since OOH affects it
      handleFsChange();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation
      handleLrdasChange();
      // Trigger FS recalculation
      handleFsChange();
    }
  }

  // Handle OOD value change
  function handleOodChange() {
    const deliveryInput = document.getElementById('delivery-date') as HTMLInputElement;
    const valueCell = document.getElementById('ood-value');
    
    if (valueCell && deliveryInput && deliveryInput.value) {
      const deliveryDate = new Date(deliveryInput.value);
      const dayOfWeek = deliveryDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      const hour = deliveryDate.getHours();
      
      let isOutOfHours = false;
      
      // Check if it's Monday-Friday (1-5) and between 5pm-8AM
      if (dayOfWeek >= 1 && dayOfWeek <= 5) {
        // Out of hours: 5pm (17:00) to 11:59pm OR 12:00am to 8am (08:00)
        if (hour >= 17 || hour < 8) {
          isOutOfHours = true;
        }
      }
      
      valueCell.textContent = isOutOfHours ? '€83.00' : '€0.00';
      updateTotal();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
    }
  }

  // Handle LSPU value change
  function handleLspuChange() {
    const pickupInput = document.getElementById('pickup-date') as HTMLInputElement;
    const valueCell = document.getElementById('lspu-value');
    
    if (valueCell && pickupInput && pickupInput.value) {
      const pickupDate = new Date(pickupInput.value);
      const dayOfWeek = pickupDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      
      // Check if it's Saturday (6)
      const isSaturday = dayOfWeek === 6;
      
      valueCell.textContent = isSaturday ? '€83.00' : '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation since LSPU affects it
      handleLrdasChange();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation
      handleLrdasChange();
    }
  }

  // Handle LSUP value change
  function handleLsupChange() {
    const pickupInput = document.getElementById('pickup-date') as HTMLInputElement;
    const valueCell = document.getElementById('lsup-value');
    
    if (valueCell && pickupInput && pickupInput.value) {
      const pickupDate = new Date(pickupInput.value);
      const dayOfWeek = pickupDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      
      // Check if it's Sunday (0)
      const isSunday = dayOfWeek === 0;
      
      valueCell.textContent = isSunday ? '€105.00' : '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation since LSUP affects it
      handleLrdasChange();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation
      handleLrdasChange();
    }
  }

  // Handle LSAT value change
  function handleLsatChange() {
    const deliveryInput = document.getElementById('delivery-date') as HTMLInputElement;
    const valueCell = document.getElementById('lsat-value');
    
    if (valueCell && deliveryInput && deliveryInput.value) {
      const deliveryDate = new Date(deliveryInput.value);
      const dayOfWeek = deliveryDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      
      // Check if it's Saturday (6)
      const isSaturday = dayOfWeek === 6;
      
      valueCell.textContent = isSaturday ? '€83.00' : '€0.00';
      updateTotal();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
    }
  }

  // Handle LSUD value change
  function handleLsudChange() {
    const deliveryInput = document.getElementById('delivery-date') as HTMLInputElement;
    const valueCell = document.getElementById('lsud-value');
    
    if (valueCell && deliveryInput && deliveryInput.value) {
      const deliveryDate = new Date(deliveryInput.value);
      const dayOfWeek = deliveryDate.getDay(); // 0 = Sunday, 1 = Monday, ..., 6 = Saturday
      
      // Check if it's Sunday (0)
      const isSunday = dayOfWeek === 0;
      
      valueCell.textContent = isSunday ? '€105.00' : '€0.00';
      updateTotal();
    } else if (valueCell) {
      valueCell.textContent = '€0.00';
      updateTotal();
    }
  }

  // Handle pickup date change - triggers OOH, LSPU, and LSUP calculations
  function handlePickupDateChange() {
    handleOohChange();
    handleLspuChange();
    handleLsupChange();
  }

  // Handle delivery date change - triggers OOD, LSAT, LSUD calculations and LRDAS2
  function handleDeliveryDateChange() {
    handleOodChange();
    handleLsatChange();
    handleLsudChange();
    // Trigger LRDAS2 recalculation since OOD, LSAT, LSUD affect it
    handleLrdas2Change();
    // Trigger FS recalculation since OOD, LSAT, LSUD affect it
    handleFsChange();
  }

  // Handle LGPSL value change
  function handleLgpslChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lgpsl-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€66.00' : '€0.00';
      
      // If LGPSL is set to YES, set GPS to NO and value to zero
      if (target.value === 'YES') {
        const gpsSelect = document.getElementById('gps-select') as HTMLSelectElement;
        const gpsValueCell = document.getElementById('gps-value');
        if (gpsSelect && gpsValueCell) {
          gpsSelect.value = 'NO';
          gpsValueCell.textContent = '€0.00';
        }
      }
      
      updateTotal();
    }
  }

  // Handle DBHC value change
  function handleDbhcChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('dbhc-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€105.00' : '€0.00';
      updateTotal();
      // Trigger LRDAS recalculation since DBHC affects it
      handleLrdasChange();
    }
  }

  // Handle DBHD value change
  function handleDbhdChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('dbhd-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€105.00' : '€0.00';
      updateTotal();
      // Trigger LRDAS2 recalculation since DBHD affects it
      handleLrdas2Change();
    }
  }

  // Handle LRDAS value change
  function handleLrdasChange() {
    const lrdasSelect = document.getElementById('lrdas-select') as HTMLSelectElement;
    const valueCell = document.getElementById('lrdas-value');
    
    if (valueCell && lrdasSelect) {
      let lrdasValue = 0;
      
      // Only calculate if user selected YES for Nordic delivery country
      if (lrdasSelect.value === 'YES') {
        // Check if any of OOH, LSPU, LSUP, DBHC are not €0.00
        const oohValue = document.getElementById('ooh-value')?.textContent || '€0.00';
        const lspuValue = document.getElementById('lspu-value')?.textContent || '€0.00';
        const lsupValue = document.getElementById('lsup-value')?.textContent || '€0.00';
        const dbhcValue = document.getElementById('dbhc-value')?.textContent || '€0.00';
        
        // Check if any of these charges are active (not €0.00)
        if (oohValue !== '€0.00' || lspuValue !== '€0.00' || lsupValue !== '€0.00' || dbhcValue !== '€0.00') {
          lrdasValue = 175;
        }
      }
      
      valueCell.textContent = lrdasValue > 0 ? `€${lrdasValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle LRDAS2 value change
  function handleLrdas2Change() {
    const lrdas2Select = document.getElementById('lrdas2-select') as HTMLSelectElement;
    const valueCell = document.getElementById('lrdas2-value');
    
    if (valueCell && lrdas2Select) {
      let lrdas2Value = 0;
      
      // Only calculate if user selected YES for Nordic delivery country
      if (lrdas2Select.value === 'YES') {
        // Check if any of OOD, LSAT, LSUD, DBHD are not €0.00
        const oodValue = document.getElementById('ood-value')?.textContent || '€0.00';
        const lsatValue = document.getElementById('lsat-value')?.textContent || '€0.00';
        const lsudValue = document.getElementById('lsud-value')?.textContent || '€0.00';
        const dbhdValue = document.getElementById('dbhd-value')?.textContent || '€0.00';
        
        // Check if any of these charges are active (not €0.00)
        if (oodValue !== '€0.00' || lsatValue !== '€0.00' || lsudValue !== '€0.00' || dbhdValue !== '€0.00') {
          lrdas2Value = 175;
        }
      }
      
      valueCell.textContent = lrdas2Value > 0 ? `€${lrdas2Value.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle LSED value change
  function handleLsedChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lsed-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€60.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LDGR value change
  function handleLdgrChange() {
    const dangerousGoodsSelect = document.getElementById('ldgr-dangerous-goods') as HTMLSelectElement;
    const mawbInput = document.getElementById('ldgr-mawb') as HTMLInputElement;
    const valueCell = document.getElementById('ldgr-value');
    
    if (valueCell && dangerousGoodsSelect && mawbInput) {
      let ldgrValue = 0;
      
      if (dangerousGoodsSelect.value === 'YES') {
        const mawbCount = parseInt(mawbInput.value) || 1; // Default to 1 if empty or invalid
        ldgrValue = 267 * mawbCount;
      }
      
      valueCell.textContent = ldgrValue > 0 ? `€${ldgrValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle FS value change
  function handleFsChange() {
    const percentageInput = document.getElementById('fs-percentage') as HTMLInputElement;
    const valueCell = document.getElementById('fs-value');
    
    if (valueCell && percentageInput) {
      const percentage = parseFloat(percentageInput.value) || 23; // Default to 23% if empty or invalid
      
      // Get values from specific charges
      const lcolValue = parseFloat((document.getElementById('lcol-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const oohValue = parseFloat((document.getElementById('ooh-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lspuValue = parseFloat((document.getElementById('lspu-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsupValue = parseFloat((document.getElementById('lsup-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const dbhcValue = parseFloat((document.getElementById('dbhc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lnfValue = parseFloat((document.getElementById('lnf-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const llwValue = parseFloat((document.getElementById('llw-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldelValue = parseFloat((document.getElementById('ldel-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const oodValue = parseFloat((document.getElementById('ood-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsatValue = parseFloat((document.getElementById('lsat-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsudValue = parseFloat((document.getElementById('lsud-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const dbhdValue = parseFloat((document.getElementById('dbhd-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const addValue = parseFloat((document.getElementById('add-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldrrValue = parseFloat((document.getElementById('ldrr-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      
      // Sum all the specified charges
      const sumOfCharges = lcolValue + oohValue + lspuValue + lsupValue + dbhcValue + lnfValue + llwValue + ldelValue + oodValue + lsatValue + lsudValue + dbhdValue + addValue + ldrrValue;
      
      // Calculate fuel surcharge as percentage of sum
      const fsValue = (percentage / 100) * sumOfCharges;
      
      valueCell.textContent = fsValue > 0 ? `€${fsValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle LSEC value change
  function handleLsecChange() {
    const percentageInput = document.getElementById('lsec-percentage') as HTMLInputElement;
    const valueCell = document.getElementById('lsec-value');
    
    if (valueCell && percentageInput) {
      const percentage = parseFloat(percentageInput.value) || 7; // Default to 7% if empty or invalid
      
      // Get values from ALL charges except FS and LSEC itself
      const lapshValue = parseFloat((document.getElementById('lapsh-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsasValue = parseFloat((document.getElementById('lsas-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lashValue = parseFloat((document.getElementById('lash-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const gpsValue = parseFloat((document.getElementById('gps-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lgpslValue = parseFloat((document.getElementById('lgpsl-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lcolValue = parseFloat((document.getElementById('lcol-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const oohValue = parseFloat((document.getElementById('ooh-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lspuValue = parseFloat((document.getElementById('lspu-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsupValue = parseFloat((document.getElementById('lsup-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const dbhcValue = parseFloat((document.getElementById('dbhc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lrdasValue = parseFloat((document.getElementById('lrdas-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const addValue = parseFloat((document.getElementById('add-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldrrValue = parseFloat((document.getElementById('ldrr-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lnfValue = parseFloat((document.getElementById('lnf-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const llwValue = parseFloat((document.getElementById('llw-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lusdValue = parseFloat((document.getElementById('lusd-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lccValue = parseFloat((document.getElementById('lcc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const liscValue = parseFloat((document.getElementById('lisc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const nuscValue = parseFloat((document.getElementById('nusc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldtfValue = parseFloat((document.getElementById('ldtf-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lbondValue = parseFloat((document.getElementById('lbond-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lexcValue = parseFloat((document.getElementById('lexc-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldelValue = parseFloat((document.getElementById('ldel-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const oodValue = parseFloat((document.getElementById('ood-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsatValue = parseFloat((document.getElementById('lsat-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsudValue = parseFloat((document.getElementById('lsud-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const dbhdValue = parseFloat((document.getElementById('dbhd-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lrdas2Value = parseFloat((document.getElementById('lrdas2-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const lsedValue = parseFloat((document.getElementById('lsed-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      const ldgrValue = parseFloat((document.getElementById('ldgr-value')?.textContent || '€0.00').replace('€', '').replace(',', '')) || 0;
      
      // Sum all charges except FS and LSEC
      const sumOfCharges = lapshValue + lsasValue + lashValue + gpsValue + lgpslValue + lcolValue + oohValue + lspuValue + lsupValue + dbhcValue + lrdasValue + addValue + ldrrValue + lnfValue + llwValue + lusdValue + lccValue + liscValue + nuscValue + ldtfValue + lbondValue + lexcValue + ldelValue + oodValue + lsatValue + lsudValue + dbhdValue + lrdas2Value + lsedValue + ldgrValue;
      
      // Calculate security surcharge as percentage of sum
      const lsecValue = (percentage / 100) * sumOfCharges;
      
      valueCell.textContent = lsecValue > 0 ? `€${lsecValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle ADD value change
  function handleAddChange() {
    const usAirportsSelect = document.getElementById('add-us-airports') as HTMLSelectElement;
    const nonUsAirportsSelect = document.getElementById('add-non-us-airports') as HTMLSelectElement;
    const valueCell = document.getElementById('add-value');
    
    if (valueCell && usAirportsSelect && nonUsAirportsSelect) {
      let addValue = 0;
      
      // Base charges for airport types
      if (usAirportsSelect.value === 'YES') {
        addValue += 167; // €167 for multiple US airports
      }
      
      if (nonUsAirportsSelect.value === 'YES') {
        addValue += 258; // €258 for multiple Non-US airports
      }
      
      // Weight-based calculation: Each increment of 22kgs after the first 22 kgs is €144
      const weightInput = document.getElementById('weight') as HTMLInputElement;
      const weight = weightInput ? parseFloat(weightInput.value) || 0 : 0;
      
      if (weight > 22) {
        const excessWeight = weight - 22;
        const weightIncrements = Math.ceil(excessWeight / 22); // Round up for any fraction
        addValue += weightIncrements * 144;
      }
      
      valueCell.textContent = addValue > 0 ? `€${addValue.toFixed(2)}` : '€0.00';
      updateTotal();
      
      // Trigger LDRR recalculation since LDRR depends on ADD
      handleLdrrChange();
      // Trigger FS recalculation since ADD affects it
      handleFsChange();
    }
  }

  // Handle LDRR value change
  function handleLdrrChange() {
    const valueCell = document.getElementById('ldrr-value');
    
    if (valueCell) {
      let ldrrValue = 0;
      
      // Check if ADD is not €0.00
      const addValue = document.getElementById('add-value')?.textContent || '€0.00';
      
      if (addValue !== '€0.00') {
        // Base charge: €149 up to 22 kg
        ldrrValue = 149;
        
        // Weight-based calculation: Each increment of 22kg after first 22kg is €25
        const weightInput = document.getElementById('weight') as HTMLInputElement;
        const weight = weightInput ? parseFloat(weightInput.value) || 0 : 0;
        
        if (weight > 22) {
          const excessWeight = weight - 22;
          const weightIncrements = Math.ceil(excessWeight / 22); // Round up for any fraction
          ldrrValue += weightIncrements * 25;
        }
      }
      
      valueCell.textContent = ldrrValue > 0 ? `€${ldrrValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle LNF value change
  function handleLnfChange() {
    const originSelect = document.getElementById('lnf-origin') as HTMLSelectElement;
    const destinationSelect = document.getElementById('lnf-destination') as HTMLSelectElement;
    const alaskaHawaiiSelect = document.getElementById('lnf-alaska-hawaii') as HTMLSelectElement;
    const valueCell = document.getElementById('lnf-value');
    
    if (valueCell && originSelect && destinationSelect && alaskaHawaiiSelect) {
      // Region values: Europe=303, UK/Switzerland/Canada=384, Middle East=418, US=337, Rest of World=577
      const regionValues: { [key: string]: number } = {
        'EUROPE': 303,
        'UK_SWISS_CANADA': 384,
        'MIDDLE_EAST': 418,
        'US': 337,
        'REST_OF_WORLD': 577
      };
      
      let lnfValue = 0;
      
      // Get values for both regions
      const originValue = regionValues[originSelect.value] || 0;
      const destinationValue = regionValues[destinationSelect.value] || 0;
      
      // Select the most expensive of the two regions
      if (originValue > 0 || destinationValue > 0) {
        lnfValue = Math.max(originValue, destinationValue);
      }
      
      // Add Alaska/Hawaii surcharge if selected
      if (alaskaHawaiiSelect.value === 'YES') {
        lnfValue += 80;
      }
      
      valueCell.textContent = lnfValue > 0 ? `€${lnfValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
    
    // Synchronize LLW region selections with LNF selections
    const llwOriginSelect = document.getElementById('llw-origin') as HTMLSelectElement;
    const llwDestinationSelect = document.getElementById('llw-destination') as HTMLSelectElement;
    
    if (llwOriginSelect && llwDestinationSelect && originSelect && destinationSelect) {
      llwOriginSelect.value = originSelect.value;
      llwDestinationSelect.value = destinationSelect.value;
      // Trigger LLW recalculation
      handleLlwChange();
    }
    
    // Also trigger NUSC recalculation since it depends on LNF destination
    handleNuscChange();
    
    // Also trigger LISC auto-calculation since it depends on LNF destination
    handleLiscAutoChange();
  }

  // Handle LLW value change
  function handleLlwChange() {
    const originSelect = document.getElementById('llw-origin') as HTMLSelectElement;
    const destinationSelect = document.getElementById('llw-destination') as HTMLSelectElement;
    const valueCell = document.getElementById('llw-value');
    
    if (valueCell && originSelect && destinationSelect) {
      // Region rates: Europe=9.85, UK/Switzerland/Canada=9.85, Middle East=12.5, US=9.85, Rest of World=15.30
      const regionRates: { [key: string]: number } = {
        'EUROPE': 9.85,
        'UK_SWISS_CANADA': 9.85,
        'MIDDLE_EAST': 12.5,
        'US': 9.85,
        'REST_OF_WORLD': 15.30
      };
      
      let llwValue = 0;
      
      // Get rates for both regions
      const originRate = regionRates[originSelect.value] || 0;
      const destinationRate = regionRates[destinationSelect.value] || 0;
      
      // Select the highest rate of the two regions
      if (originRate > 0 || destinationRate > 0) {
        const highestRate = Math.max(originRate, destinationRate);
        
        // Get weight from shipment details
        const weightInput = document.getElementById('weight') as HTMLInputElement;
        const weight = weightInput ? parseFloat(weightInput.value) || 0 : 0;
        
        // Calculate: (weight - 1) * highest rate
        if (weight > 1) {
          llwValue = (weight - 1) * highestRate;
        }
      }
      
      valueCell.textContent = llwValue > 0 ? `€${llwValue.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }

  // Handle LUSD value change
  function handleLusdChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lusd-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€52.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LCC value change
  function handleLccChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lcc-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€135.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LISC value change
  function handleLiscChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lisc-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€113.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LISC automatic calculation based on LNF destination
  function handleLiscAutoChange() {
    const lnfDestinationSelect = document.getElementById('lnf-destination') as HTMLSelectElement;
    const valueCell = document.getElementById('lisc-value');
    
    if (valueCell && lnfDestinationSelect) {
      // Check if LNF destination is US
      const isDestinationUS = lnfDestinationSelect.value === 'US';
      valueCell.textContent = isDestinationUS ? '€113.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle NUSC value change
  function handleNuscChange() {
    const lnfDestinationSelect = document.getElementById('lnf-destination') as HTMLSelectElement;
    const valueCell = document.getElementById('nusc-value');
    
    if (valueCell && lnfDestinationSelect) {
      // Check if LNF destination is US
      const isDestinationUS = lnfDestinationSelect.value === 'US';
      valueCell.textContent = isDestinationUS ? '€238.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LDTF value change
  function handleLdtfChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('ldtf-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€53.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LBOND value change
  function handleLbondChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lbond-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€140.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LEXC value change
  function handleLexcChange(event: Event) {
    const target = event.target as HTMLSelectElement;
    const valueCell = document.getElementById('lexc-value');
    if (valueCell) {
      valueCell.textContent = target.value === 'YES' ? '€238.00' : '€0.00';
      updateTotal();
    }
  }

  // Handle LDEL value change
  function handleLdelChange() {
    const regionSelect = document.getElementById('ldel-region') as HTMLSelectElement;
    const milesInput = document.getElementById('ldel-miles') as HTMLInputElement;
    const valueCell = document.getElementById('ldel-value');
    
    if (valueCell && regionSelect && milesInput) {
      const region = regionSelect.value;
      const miles = parseFloat(milesInput.value) || 0;
      let value = 0;
      
      // First 15 miles are free for all regions
      if (miles > 15) {
        const chargableMiles = miles - 15;
        
        switch(region) {
          case 'US':
            value = chargableMiles * 2.85; // €2.85 per mile after 15 miles
            break;
          case 'EU-UK':
            value = chargableMiles * 3.2; // €3.20 per mile after 15 miles
            break;
          case 'JP':
            value = chargableMiles * 4.1; // €4.10 per mile after 15 miles
            break;
          case 'ROW':
            value = chargableMiles * 4.1; // €4.10 per mile after 15 miles
            break;
          default:
            value = 0;
        }
      }
      
      valueCell.textContent = value > 0 ? `€${value.toFixed(2)}` : '€0.00';
      updateTotal();
    }
  }
</script>

<div class="min-h-screen bg-gray-50">
  <!-- Header with Sterling logo -->
  <header class="bg-white border-b border-gray-200 sticky top-0 z-10">
    <div class="max-w-7xl mx-auto px-4 py-3 flex items-center">
      <img 
        src={sterlingImage} 
        alt="Sterling" 
        class="h-12 w-auto"
      />
    </div>
  </header>

  <!-- Main content -->
  <div class="max-w-4xl mx-auto px-4 py-8">
    <!-- Hero Section -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-8 mb-6">
      <div class="text-center">
        <!-- Big Blue Header -->
        <h1 class="text-3xl font-bold text-gray-900 mb-1">
          Welcome to
        </h1>
        
        <!-- Curved Text Around Image -->
        <div class="relative mb-8">
          <!-- SVG for curved text -->
          <svg viewBox="0 0 500 250" class="w-[450px] h-[225px] mx-auto">
            <defs>
              <!-- Define the path for the text to follow (more curved semi-circle) -->
              <path id="curve" d="M 60 200 Q 250 30 440 200" fill="none" />
            </defs>
            
            <!-- The curved text -->
            <text class="fill-blue-500 text-3xl font-semibold tracking-wide">
              <textPath href="#curve" startOffset="50%" text-anchor="middle">
                Peter's Quoting Tool
              </textPath>
            </text>
          </svg>
          
          <!-- Larger Circular Image with more space -->
          <div class="absolute -bottom-20 left-1/2 transform -translate-x-1/2">
            <div class="w-48 h-48 rounded-full overflow-hidden ring-4 ring-white shadow-xl bg-gray-100 flex items-center justify-center hover-image-container">
              <!-- Peter's image -->
              <img 
                src={peterImage} 
                alt="Peter" 
                class="w-full h-full object-cover hover-image"
                on:error={handleImageError}
              />
              <!-- Fallback placeholder -->
              <div class="w-full h-full bg-blue-100 flex items-center justify-center text-blue-600 font-bold text-4xl hidden">
                P
              </div>
            </div>
          </div>
        </div>
        
        <!-- Subheader -->
        <div class="mt-24 pt-4">
          <h2 class="text-xl font-medium text-gray-600 mb-8">
            As if the man himself provided you a quote.
          </h2>
        </div>
      </div>
    </div>
    
    <!-- Account Selection Card -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6 mb-6">
      <div class="flex items-center justify-center space-x-4">
        <label for="account-select" class="text-gray-700 font-medium">Choose Account:</label>
        <select id="account-select" class="px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 font-medium bg-white text-gray-900">
          <option value="EUR1" style="color: #dc2626; font-weight: 600;" selected>EUR1</option>
        </select>
      </div>
    </div>
    
    <!-- Shipment Details Card -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 p-6 mb-6">
      <h3 class="text-lg font-semibold text-gray-900 mb-4">Shipment Details</h3>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <!-- Weight -->
        <div>
          <label for="weight" class="block text-sm font-medium text-gray-700 mb-2">Weight (kg)</label>
          <input 
            type="number" 
            id="weight" 
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-gray-900" 
            placeholder="Enter weight in kg"
            on:input={handleWeightChange}
          />
        </div>
        
        <!-- Origin Airport -->
        <div>
          <label for="origin-airport" class="block text-sm font-medium text-gray-700 mb-2">Origin Airport</label>
          <input 
            type="text" 
            id="origin-airport" 
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-gray-900" 
            placeholder="e.g., LAX, JFK, LHR"
          />
        </div>
        
        <!-- Destination Airport -->
        <div>
          <label for="destination-airport" class="block text-sm font-medium text-gray-700 mb-2">Destination Airport</label>
          <input 
            type="text" 
            id="destination-airport" 
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-gray-900" 
            placeholder="e.g., CDG, NRT, DXB"
          />
        </div>
      </div>
      
      <!-- Second row for dates -->
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-4">
        <!-- Pickup Date -->
        <div>
          <label for="pickup-date" class="block text-sm font-medium text-gray-700 mb-2">Pickup Date</label>
          <input 
            type="datetime-local" 
            id="pickup-date" 
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-gray-900"
            on:input={handlePickupDateChange}
          />
        </div>
        
        <!-- Delivery Date -->
        <div>
          <label for="delivery-date" class="block text-sm font-medium text-gray-700 mb-2">Delivery Date</label>
          <input 
            type="datetime-local" 
            id="delivery-date" 
            class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-gray-900"
            on:input={handleDeliveryDateChange}
          />
        </div>
      </div>
    </div>
    
    <!-- Quote Form Card -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden">
      <div class="px-6 py-4 border-b border-gray-200">
        <h3 class="text-lg font-semibold text-gray-900">Service Options</h3>
      </div>
      
      <div class="overflow-x-auto">
        <table class="w-full">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Code</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Description</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Input</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Value</th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <!-- LAPSH -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LAPSH</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Airport Special Handling - Sodexi) - <span class="text-blue-500 font-medium">vendorcode</span> = "SOD001"</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLapshChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lapsh-value">€0.00</td>
            </tr>
            
            <!-- LSAS -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSAS</td>
              <td class="px-6 py-4 text-sm text-gray-700">Is the job "SAS"</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLsasChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsas-value">€0.00</td>
            </tr>
            
            <!-- LASH -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LASH</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Airline Special Handling) Use tiers to determine corresponding tier</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lash-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLashChange}>
                  <option value="NONE" selected>Select Tier</option>
                  <option value="TIER1">Tier 1 Level (Up to 22 kgs) - €184</option>
                  <option value="TIER2">Tier 2 Level (Up to 22 kgs) - €248</option>
                  <option value="TIER3">Tier 3 Level (Up to 22 kgs) - €423</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lash-value">€0.00</td>
            </tr>
            
            <!-- GPS -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">GPS</td>
              <td class="px-6 py-4 text-sm text-gray-700">(GPS Tracking Unit) Is there GPS?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="gps-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleGpsChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="gps-value">€0.00</td>
            </tr>
            
            <!-- LGPSL -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LGPSL</td>
              <td class="px-6 py-4 text-sm text-gray-700">Is there a Roambee attached?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lgpsl-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLgpslChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lgpsl-value">€0.00</td>
            </tr>
            
            <!-- LCOL -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LCOL</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Collection)</td>
              <td class="px-6 py-4">
                <div class="space-y-2">
                  <select id="lcol-region" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLcolChange}>
                    <option value="">Select Region</option>
                    <option value="US">US - €2.36/mile</option>
                    <option value="EU-UK">EU-UK - €3.70/mile</option>
                    <option value="JP">Japan - €2.10/mile</option>
                    <option value="ROW">Rest of World - €4.78/mile</option>

                  </select>
                  <input 
                    type="number" 
                    id="lcol-miles" 
                    class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" 
                    placeholder="Shipper miles (first 15 free)" 
                    on:input={handleLcolChange}
                  />
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lcol-value">€0.00</td>
            </tr>
            
            <!-- OOH -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">OOH</td>
              <td class="px-6 py-4 text-sm text-gray-700">Out of Hours Collection If Pick up Time is between 5pm-8AM Mon-Fri</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Pickup Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ooh-value">€0.00</td>
            </tr>
            
            <!-- LSPU -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSPU</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Saturday Pickup) €83 if pickupdate falls on a Saturday</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Pickup Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lspu-value">€0.00</td>
            </tr>
            
            <!-- LSUP -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSUP</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Sunday Pickup) €105 if pickupdate falls on a Sunday</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Pickup Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsup-value">€0.00</td>
            </tr>
            
            <!-- DBHC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">DBHC</td>
              <td class="px-6 py-4 text-sm text-gray-700">Holiday Pickup?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="dbhc-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleDbhcChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="dbhc-value">€0.00</td>
            </tr>
            
            <!-- LRDAS -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LRDAS</td>
              <td class="px-6 py-4 text-sm text-gray-700">Is The PickUp Country DENMARK, NORWAY, SWEDEN, FINLAND OR ICELAND?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lrdas-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLrdasChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lrdas-value">€0.00</td>
            </tr>
            
            <!-- ADD -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">ADD</td>
              <td class="px-6 py-4 text-sm text-gray-700">Transshipment - Multiple Airports in the route?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="flex flex-col space-y-2">
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Multiple US Airports?</span>
                    <select id="add-us-airports" class="w-24 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleAddChange}>
                      <option value="NO" selected>NO</option>
                      <option value="YES">YES</option>
                    </select>
                  </div>
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Multiple Non-US Airports?</span>
                    <select id="add-non-us-airports" class="w-24 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleAddChange}>
                      <option value="NO" selected>NO</option>
                      <option value="YES">YES</option>
                    </select>
                  </div>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="add-value">€0.00</td>
            </tr>
            
            <!-- LDRR -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LDRR</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Airline Retrieval) - System Input - If ADD is not 0, then €149 up to 22 kg from Weight, each increment of 22kg is another €25</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">System calculates based on ADD and Weight</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ldrr-value">€0.00</td>
            </tr>
            
            <!-- LNF -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LNF</td>
              <td class="px-6 py-4 text-sm text-gray-700">Next Flight Out Door to Door Service - Select most expensive of the two regions</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="space-y-2">
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Origin Region:</span>
                    <select id="lnf-origin" class="w-40 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLnfChange}>
                      <option value="" selected>Select Region</option>
                      <option value="EUROPE">Europe - €303</option>
                      <option value="UK_SWISS_CANADA">UK, Switzerland & Canada - €384</option>
                      <option value="MIDDLE_EAST">Middle East - €418</option>
                      <option value="US">US - €337</option>
                      <option value="REST_OF_WORLD">Rest of World - €577</option>
                    </select>
                  </div>
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Destination Region:</span>
                    <select id="lnf-destination" class="w-40 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLnfChange}>
                      <option value="" selected>Select Region</option>
                      <option value="EUROPE">Europe - €303</option>
                      <option value="UK_SWISS_CANADA">UK, Switzerland & Canada - €384</option>
                      <option value="MIDDLE_EAST">Middle East - €418</option>
                      <option value="US">US - €337</option>
                      <option value="REST_OF_WORLD">Rest of World - €577</option>
                    </select>
                  </div>
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Alaska/Hawaii:</span>
                    <select id="lnf-alaska-hawaii" class="w-24 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLnfChange}>
                      <option value="NO" selected>NO</option>
                      <option value="YES">YES</option>
                    </select>
                  </div>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lnf-value">€0.00</td>
            </tr>
            
            <!-- LLW -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LLW</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Weight Charges)</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="space-y-2">
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Origin Region:</span>
                    <select id="llw-origin" class="w-40 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLlwChange}>
                      <option value="" selected>Select Region</option>
                      <option value="EUROPE">Europe - €9.85</option>
                      <option value="UK_SWISS_CANADA">UK, Switzerland & Canada - €9.85</option>
                      <option value="MIDDLE_EAST">Middle East - €12.50</option>
                      <option value="US">US - €9.85</option>
                      <option value="REST_OF_WORLD">Rest of World - €15.30</option>
                    </select>
                  </div>
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Destination Region:</span>
                    <select id="llw-destination" class="w-40 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLlwChange}>
                      <option value="" selected>Select Region</option>
                      <option value="EUROPE">Europe - €9.85</option>
                      <option value="UK_SWISS_CANADA">UK, Switzerland & Canada - €9.85</option>
                      <option value="MIDDLE_EAST">Middle East - €12.50</option>
                      <option value="US">US - €9.85</option>
                      <option value="REST_OF_WORLD">Rest of World - €15.30</option>
                    </select>
                  </div>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="llw-value">€0.00</td>
            </tr>
            
            <!-- LUSD -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LUSD</td>
              <td class="px-6 py-4 text-sm text-gray-700">Prepare International Paperwork - Not US to US?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lusd-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLusdChange}>
                  <option value="NO">NO</option>
                  <option value="YES" selected>YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lusd-value">€52.00</td>
            </tr>
            
            <!-- LCC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LCC</td>
              <td class="px-6 py-4 text-sm text-gray-700">Customs Clearance - Is origin and destination NOT EU - EU ?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lcc-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLccChange}>
                  <option value="NO">NO</option>
                  <option value="YES" selected>YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lcc-value">€135.00</td>
            </tr>
            
            <!-- LISC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LISC</td>
              <td class="px-6 py-4 text-sm text-gray-700">US Import Service Charge - Is Destination Country US?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">System Input</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lisc-value">€0.00</td>
            </tr>
            
            <!-- NUSC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">NUSC</td>
              <td class="px-6 py-4 text-sm text-gray-700">US Import Clearance</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">System Input</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="nusc-value">€0.00</td>
            </tr>
            
            <!-- LDTF -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LDTF</td>
              <td class="px-6 py-4 text-sm text-gray-700">Documentation Handover?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="ldtf-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLdtfChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ldtf-value">€0.00</td>
            </tr>
            
            <!-- LBOND -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LBOND</td>
              <td class="px-6 py-4 text-sm text-gray-700">Transit Bond?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lbond-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLbondChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lbond-value">€0.00</td>
            </tr>
            
            <!-- LEXC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LEXC</td>
              <td class="px-6 py-4 text-sm text-gray-700">Export Clearance, Is Shipper Country - China, Mexico, Colombia, Brazil, Switzerland, Turkey or Qatar?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lexc-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLexcChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lexc-value">€0.00</td>
            </tr>
            
            <!-- LDEL -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LDEL</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Delivery)</td>
              <td class="px-6 py-4">
                <div class="space-y-2">
                  <select id="ldel-region" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLdelChange}>
                    <option value="">Select Region</option>
                    <option value="US">US - €2.85/mile</option>
                    <option value="EU-UK">EU-UK - €3.20/mile</option>
                    <option value="JP">Japan - €4.10/mile</option>
                    <option value="ROW">Rest of World - €4.10/mile</option>
                  </select>
                  <input 
                    type="number" 
                    id="ldel-miles" 
                    class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" 
                    placeholder="Consignee miles (first 15 free)" 
                    on:input={handleLdelChange}
                  />
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ldel-value">€0.00</td>
            </tr>
            
            <!-- OOD -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">OOD</td>
              <td class="px-6 py-4 text-sm text-gray-700">Out of Hours Delivery If Delivery Time is between 5pm-8AM Mon-Fri</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Delivery Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ood-value">€0.00</td>
            </tr>
            
            <!-- LSAT -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSAT</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Saturday Delivery) €83 if delivery date falls on a Saturday</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Delivery Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsat-value">€0.00</td>
            </tr>
            
            <!-- LSUD -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSUD</td>
              <td class="px-6 py-4 text-sm text-gray-700">(Sunday Delivery) €105 if delivery date falls on a Sunday</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <span class="text-sm text-gray-500 italic">Uses Delivery Date from Shipment Details</span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsud-value">€0.00</td>
            </tr>
            
            <!-- DBHD -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">DBHD</td>
              <td class="px-6 py-4 text-sm text-gray-700">Holiday Delivery?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="dbhd-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleDbhdChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="dbhd-value">€0.00</td>
            </tr>
            
            <!-- LRDAS2 -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LRDAS2</td>
              <td class="px-6 py-4 text-sm text-gray-700">Is The Delivery Country DENMARK, NORWAY, SWEDEN, FINLAND OR ICELAND?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lrdas2-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLrdas2Change}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lrdas2-value">€0.00</td>
            </tr>
            
            <!-- LSED -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSED</td>
              <td class="px-6 py-4 text-sm text-gray-700">SED Filling - AES?</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <select id="lsed-select" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLsedChange}>
                  <option value="NO" selected>NO</option>
                  <option value="YES">YES</option>
                </select>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsed-value">€0.00</td>
            </tr>
            
            <!-- LDGR -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LDGR</td>
              <td class="px-6 py-4 text-sm text-gray-700">Dangerous Goods Fee</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="space-y-2">
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">Dangerous Goods?</span>
                    <select id="ldgr-dangerous-goods" class="w-24 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" on:change={handleLdgrChange}>
                      <option value="NO" selected>NO</option>
                      <option value="YES">YES</option>
                    </select>
                  </div>
                  <div class="flex items-center justify-between">
                    <span class="text-xs text-gray-600">How many MAWB?</span>
                    <input 
                      type="number" 
                      id="ldgr-mawb" 
                      class="w-24 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" 
                      value="1"
                      min="1"
                      on:input={handleLdgrChange}
                    />
                  </div>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="ldgr-value">€0.00</td>
            </tr>
            
            <!-- FS -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">FS</td>
              <td class="px-6 py-4 text-sm text-gray-700">Fuel Surcharge</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="flex items-center space-x-2">
                  <input 
                    type="number" 
                    id="fs-percentage" 
                    class="w-20 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" 
                    value="23"
                    min="0"
                    step="0.1"
                    on:input={handleFsChange}
                  />
                  <span class="text-sm text-gray-600">%</span>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="fs-value">€0.00</td>
            </tr>
            
            <!-- LSEC -->
            <tr class="hover:bg-gray-50 transition-colors">
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">LSEC</td>
              <td class="px-6 py-4 text-sm text-gray-700">Security Surcharge</td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="flex items-center space-x-2">
                  <input 
                    type="number" 
                    id="lsec-percentage" 
                    class="w-20 px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 text-sm" 
                    value="7"
                    min="0"
                    step="0.1"
                    on:input={handleLsecChange}
                  />
                  <span class="text-sm text-gray-600">%</span>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900" id="lsec-value">€0.00</td>
            </tr>
          </tbody>
        </table>
      </div>
      
      <!-- Running Total -->
      <div class="px-6 py-4 bg-gray-50 border-t border-gray-200">
        <div class="flex justify-between items-center">
          <span class="text-lg font-semibold text-gray-900">Total Quote</span>
          <span class="text-2xl font-bold text-blue-600" id="running-total">€187.00</span>
        </div>
        
        <!-- Disclaimer -->
        <div class="mt-4 pt-3 border-t border-gray-300">
          <p class="text-xs text-gray-500 italic">
            * Quote not Inclusive of Out of Hours Clearance, Wait Time, Pickup Or Delivery Attempts, VAT and Duty Charges
          </p>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  /* Instagram-inspired styling */
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  }
  
  /* Smooth transitions for all interactive elements */
  input, select, button {
    transition: all 0.2s ease;
  }
  
  /* Focus states */
  input:focus, select:focus {
    outline: none;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }
  
  /* Card hover effects */
  .bg-white:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
    transition: box-shadow 0.2s ease;
  }
  
  /* SVG text styling */
  svg text {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  }
  
  /* Peter's image hover effect */
  .hover-image-container {
    transition: all 0.3s ease;
  }
  
  .hover-image-container:hover {
    transform: scale(1.1);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
    z-index: 10;
  }
  
  .hover-image {
    transition: all 0.3s ease;
  }
  
  .hover-image-container:hover .hover-image {
    transform: scale(1.05);
  }
</style>
